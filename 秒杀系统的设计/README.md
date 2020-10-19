### 设计方案
　　概括为动静分离、集群负载均衡、缓存、限流、线程池。

- 动静分离，Nginx + CDN 实现；
- 集群负载均衡，对请求进行负载均衡，分散流量；
- 缓存，多级缓存；
    1. 查询通过本机缓存 + redis；
    2. 扣减库存先在 redis 中扣减，后同步到数据库；
- 限流，对接口请求限流，比如令牌桶、漏桶、信号量等；
- 线程池，多个线程来并发处理接口请求。

### 流程
　　扣减库存放在 redis 中，通过中间件发送消息，最后消费端消费消息，同步到数据库。过程为客户端下单 -> redis 扣减库存，发送消息到 RocketMQ -> RocketMQ 发送到后端，处理订单，扣减数据库的库存。

- 活动开始前，将数据库的存储信息，比如商品数量等，同步到 redis 中。注意，该商品需要下架，防止在活动开始前，有人下订单，导致 redis 和数据库的商品数量不一致；
- 将商品、活动的查询先预载，放到缓存中；
- 请求进来，Nginx 进行负载均衡；
- 请求限流，假设有 20 个商品，进来 10W 个请求，就不必要，使用 RateLimiter 限制只能进来 2000 个请求等
- 客户端下单请求，线程池并发处理；
- 线程池执行任务，调用中间件，发送事务型消息，为 prepared，执行创建订单逻辑（事务注解修饰）；
    1. 在 redis 中扣减库存；
    2. 生成订单号，创建订单，插入到数据库中；
    3. 返回订单对象，消息状态由 prepared 改为 commit；
- 中间件检测到消息结果为 commit，将消息发送到消费端；
- 消费端在数据库中扣减库存，返回 ACK 给 MQ，完成一次下单操作。

### 动静分离
　　前端的静态资源放在 Nginx 中，还可以在优化将其放在 CDN 服务器上，如没有获取到，再从 Nginx 上获取。

### 分布式扩展
　　Nginx 反向代理进行负载均衡，将流量请求分散到多台机器上，解决单点故障和降低单个服务器的压力。<br />
　　同时，优化 Nginx 与后端服务器的连接，默认是短连接，通过修改 nginx.conf 将其改为长连接。

```conf
http {
    upstream backend {
        # 反向代理，两台机器进行负载
        server 192.168.0.1：8080 weight=1 max_fails=2 fail_timeout=30s;
        server 192.168.0.2：8080 weight=1 max_fails=2 fail_timeout=30s;
        # 长连接时间
        keepalive 300;       
    }  
 
    server {
        listen 8080 default_server;
        server_name "";
    
        location / {
            proxy_pass http://backend;
            # 设置 http 版本为 1.1
            proxy_http_version 1.1; 
            # 设置 Connection 为长连接，默认为 no    
            proxy_set_header Connection "";
        }
    }
}
```

### 多级缓存
　　使用 redis + 本机缓存 guava cache，先到本地缓存中获取，没有再到 redis 中获取，最后才到数据库中进行获取。<br />
　　比如，在查询秒杀活动是否存在时，可将秒杀活动 ID 先进行缓存。由于活动具有时效性，所以在缓存类中加一个清除接口，即清除过期的活动信息缓存。

### 限流
　　假设秒杀商品只有 20 个，可以限制 2000 个请求进来，如下代码。使用 RateLimiter 限流，有令牌桶和漏桶，另一种使用信号量 semaphore 限流。

```java
private RateLimiter rateLimiter;

@PostConstruct
public void init() {
    // 初始化，使用限流器限流
    rateLimiter = RateLimiter.create(100);
}

// 限流，超过 2000 个请求，则返回异常
if (!rateLimiter.tryAcquire()) {
    // 抛出异常
}
```

### SpringBoot 内嵌 Tomcat 优化
　　根据硬件设备和业务需求，调整 Tomcat 的最大等待队列、最大可连接数、最大工作线程数、最小工作线程数等。默认情况，最大可连接数为 10000，默认最大工作线程数为 200，默认最小工作线程数为 10，最大等待队列为 100。当请求超过最大工作线程数 + 最大等待队列时，即 200 + 100，会拒绝处理。<br />
　　一般调整最大工作线程数、最小工作线程数、最大等待队列等。当然不是越大越好，一是受内存限制，二是线程越多，CPU 上下文切换开销越大。最小工作线程数根据正常情况下的请求来设置，可在 application.properties 里面进行设置。

```properties
server.tomcat.accept-count=1000

server.tomcat.max-threads=800

server.tomcat.min-spare-threads=100

server.tomcat.max-connections=10000
```

#### 网络优化
　　关于内嵌 Tomcat 的网络优化，通过 WebServerFactoryCustomizer&lt;ConfigurableWebServerFactory&gt; 接口来设置，比如自定义 
KeepAlive，保持长连接，避免重复建立连接，减少三次握手的和四次挥手的开销。

- keepAliveTimeOut，多少毫秒后不响应的断开 keepalive；
- maxKeepAliveRequests，多少次请求后 keepalive 断开失效。

```java
@Component
public class WebServerConfiguration implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        //使用对应工厂类提供给我们的接口，定制化Tomcat connector
        ((TomcatServletWebServerFactory) factory).addConnectorCustomizers(new TomcatConnectorCustomizer() {
            @Override
            public void customize(Connector connector) {
                Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
                //定制化KeepAlive Timeout为30秒
                protocol.setKeepAliveTimeout(30000);
                //10000个请求则自动断开
                protocol.setMaxKeepAliveRequests(10000);
            }
        });
    }
}
```

### 线程池并发发送下单消息
　　当有多个请求进来时，使用线程池并发处理，创建订单。这里，**通过线程池来调用中间件发送事务型消息，该消息会调用真正的下单方法，进行下单。** 这样当该消息发送失败时，会回滚。线程池中的处理逻辑如下：

- 首先，在数据库中插入一条记录，为库存流水的初始状态；
    1. 库存流水状态，包含字段商品 ID、商品数量、log ID、状态 status，其中 status 包含初始状态 1、下单扣减库存成功 2、下单回滚 3。
    2. 如果已在数据库中初始化库存流水的初始状态，但没调用中间件发送消息，就挂了，这时事务消息为 UNKNOWN 状态，使用库存流水状态来解决；
    3. 发送事务型消息下订单前，先初始化库存流水的状态，下单成功，在修改库存流水的状态。
- 调用中间件，发送消息型事务消息。

```java
    // 创建固定的线程池，线程数量为 20
    ExecutorService executorService = Executors.newFixedThreadPool(20);

    public CommonReturnType createOrder(@RequestParam(name = "itemId") Integer itemId,
                                        @RequestParam(name = "amount") Integer amount) throws BizException {

        // 使用线程池，并发发送事务型消息，创建订单
        Future<Object> future = executorService.submit(new Callable<Object>() {
            @Override
            public Object call() throws Exception {
                // 加入库存流水 init 状态，该状态为初始状态
                String stockLogId = itemService.initStockLog(itemId, amount);
                // 消息事务处理方法
                if (!mqProducer.transactionAsyncReduceStock(userModel.getId(), itemId, promoId, amount, stockLogId)) {
                    throw new BizException(EmBizError.UNKNOWN_ERROR, "下单失败");
                }
                return null;
            }
        });

        try {
            future.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
            throw new BizException(EmBizError.UNKNOWN_ERROR);
        } catch (ExecutionException e) {
            e.printStackTrace();
            throw new BizException(EmBizError.UNKNOWN_ERROR);
        }
        return CommonReturnType.create(null);
    }
}
```

### 中间件事务设置
　　让中间件发送事务消息，该消息发送到消息队里里，处于 prepared 状态。broker 接收到该消息后，不会把这条消息给消费者进行消费。<br />
　　处于 prepared 状态的消息，会先执行 executeLocalTransaction 方法，该方法为创建订单流程。**消费端需要根据 executeLocalTransaction 返回的结果，来判断是消费消息还是不消费。即创建订单订单成功，则消费端会消费，否则就不消费。** <br />
　　executeLocalTransaction 方法的业务逻辑。

- 正常发送消息成功，broker 接收到 prepared 状态的消息，会调用 orderService#createOrder 方法创建订单；
    1. orderService#createOrder 方法使用 @Transactional 修饰，失败会进行回滚；
    2. 同时会触发中间件的回滚，设置库存流水状态为回滚，同时返回 LocalTransactionState.ROLLBACK_MESSAGE，这样消费端就不消费。
- 消费端检查到消息结果为 LocalTransactionState.COMMIT_MESSAGE，即创建订单成功，执行消息提交，消费端可消费该消息，在数据库中进行库存扣减。

```java
    private TransactionMQProducer transactionMQProducer;

    @Value("${mq.nameserver.addr}")
    private String nameAddr;

    @Autowired
    private OrderService orderService;

    @Autowired
    private StockLogDOMapper stockLogDOMapper;

    @PostConstruct
    public void init() throws MQClientException {
        // 启动生产者
        producer = new DefaultMQProducer("producer_group");
        producer.setNamesrvAddr(nameAddr);
        producer.start();
        // 启动事务
        transactionMQProducer = new TransactionMQProducer("transaction_producer_group");
        transactionMQProducer.setNamesrvAddr(nameAddr);
        transactionMQProducer.start();

        // 事务设置
        transactionMQProducer.setTransactionListener(new TransactionListener() {
            @Override
            public LocalTransactionState executeLocalTransaction(Message message, Object args) {
                // 接收消息，获取 args，为 map，包含商品 ID、用户 ID 等
                Integer itemId = (Integer) ((Map) args).get("itemId");
                Integer promoId = (Integer) ((Map) args).get("promoId");
                Integer userId = (Integer) ((Map) args).get("userId");
                Integer amount = (Integer) ((Map) args).get("amount");
                String stockLogId = (String) ((Map) args).get("stockLogId");

                try {
                    // 创建订单
                    orderService.createOrder(userId, itemId, promoId, amount, stockLogId);
                } catch (BizException e) {
                    e.printStackTrace();
                    // 发生异常，createOrder 使用 @Transactional 修饰，已回滚。这时要回滚事务型消息，
                    // 设置库存流水 stockLog 状态为回滚状态 3，如果这里挂了，只能根据日志，来人工设置
                    StockLogDO stockLogDO = stockLogDOMapper.selectByPrimaryKey(stockLogId);
                    stockLogDO.setStatus(3);
                    stockLogDOMapper.updateByPrimaryKeySelective(stockLogDO);
                    return LocalTransactionState.ROLLBACK_MESSAGE;
                }
                // 创建订单成功，执行消息提交，消费端可消费该消息，在数据库中进行库存扣减
                return LocalTransactionState.COMMIT_MESSAGE;
            }

            /**
             * 检查消息的状态，根据订单是否创建成功，来判断是否提交消息
             * 
             * 1. 从数据库中获取存储流水状态；
             * 2. 为 2，表示创建订单成功，提交事务型消息；
             * 3. 为 1，表示还在初始状态，没进行订单操作，消息继续保持 prepare 状态，不提交消息。
             */
            @Override
            public LocalTransactionState checkLocalTransaction(MessageExt message) {
                String jsonString = new String(message.getBody());
                Map<String, Object> map = JSON.parseObject(jsonString, Map.class);
                String stockLogId = (String) map.get("stockLogId");
                // 从数据库中获取存储流水状态
                StockLogDO stockLogDO = stockLogDOMapper.selectByPrimaryKey(stockLogId);
                if (stockLogDO == null)
                    return LocalTransactionState.UNKNOW;
                // 创建订单成功，等着异步扣减库存，于是提交事务型消息
                if (stockLogDO.getStatus() == 2) {
                    return LocalTransactionState.COMMIT_MESSAGE;
                    // 为初始状态，还没进行订单操作，消息继续保持 prepare 状态
                } else if (stockLogDO.getStatus() == 1) {
                    return LocalTransactionState.UNKNOW;
                }
                return LocalTransactionState.ROLLBACK_MESSAGE;
            }
        });
    }
```

### 异步减库存，缓存扣减下单
　　有如下几步，任何一步抛异常，即会回滚。

- 参数校验；
- 落单减库存，在 redis 库存中扣减，redis 回滚是将库存数量 + 1；
- 创建订单，生成流水号，并入库；
- 更新库存流水状态为 2，表示创建订单成功；
- 返回订单对象。

```java
    @Override
    @Transactional
    public OrderModel createOrder(Integer userId, Integer itemId, Integer promoId, Integer amount, String stockLogId) throws BizException {

        // 参数校验
        // 落单减库存，在 redis 库存中扣减，redis 回滚是将库存数量 + 1
        // 创建订单，生成流水号，并入库
        // 更新库存流水状态为 2，表示创建订单成功
        // 返回订单对象
    }
```

### 数据库扣减库存
　　当下订单成功，MQ 会发送消息到消费端，消费端在数据库中进行库存扣减。为保证消费成功，当同步数据库成功，才发送 ACK 到 MQ。

### 生产者丢失

- 程序发送失败抛异常，没有进行重试；
- 发送成功，但是由于网络原因，MQ 没有收到。

#### 解决方案
　　在异步发送中，通过回调通知 + 本地消息表的形式来解决，**使用 RocketMQ 的事务型消息设置。** <br />

- controller 中调用生产者发送一条事务型消息，在事务性消息中会执行下单操作，该消息状态为 prepared；
    1. 如果下单逻辑执行成功，则消费端会进行消费，即到数据库中扣减库存；
    2. 如果下单失败或异常，则会进行事务回滚。
- JOB 轮询，对超过一定时间还未发送成功的消息进行重试；
- 如果超过一定次数还未发送成功，则进行人工介入。

### MQ 丢失
　　持久化到磁盘可解决，即刷盘，又分为同步刷盘和异步刷盘。默认异步刷盘，可能导致消息没刷到硬盘上就丢失，可使用同步刷盘来保证消息不丢失。

### 消费者丢失
　　消费者收到消息后，没消费就挂掉了，MQ 以为消费了。使用 ACK 确认机制，当消费者消费完消息后，才发送 ACK 给 MQ。如果 MQ 没收到 ACK，则进行重发。

