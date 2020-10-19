
### Catalina#start
　　使用 Digester 解析 server.xml，创建 Server 组件的实例和各种子组件（GlobalNamingResources、Listener、Service、Executor、Connector）的实例。

- 如果 Server 为空，则调用 [Catalina#load](https://github.com/martin-1992/Tomcat-Notes/blob/master/Tomcat%20%E7%9A%84%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B/Catalina%23load.md)，解析 server.xml 创建 Server 组件和各种子组件，进行初始化 init()；
- 再次调用 getServer 判断 Server 组件是否创建成功，为空创建失败，则返回；
- 调用 Server 的 start 方法，启动 Server，start() 同 init() 方法一样，为抽象方法。该抽象方法中的 startInternal 默认由子类 [StandardServer#startInternal](#https://github.com/martin-1992/Tomcat-Notes/blob/master/Tomcat%20%E7%9A%84%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B/StandardServer%23startInternal.md)实现，该方法会调用各 service 组件的 start 方法，然后各 service 组件继续调用各子组件如 executor、connector 的 start 方法（在 Catalina#load 已对各组件进行创建和初始化），可参考 [StandardService#startInternal](https://github.com/martin-1992/Tomcat-Notes/blob/master/Tomcat%20%E7%9A%84%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B/StandardService%23startInternal.md)；
- 注册一个关闭钩子，如果使用钩子，则调用 Server 的 stop 方法，释放所有资源；
- 使用 await 方法监听停止请求。
    
```java
    public void start() {
        // 如果 Server 为空，则使用 Digester 解析 xml 文件创建 Server 和各种组件（GlobalNamingResources、Listener、
        // Service、Executor、Connector），Tomcat 使用多层容器设计，最外层为 Server，可理解为 Tomcat 实例
        if (getServer() == null) {
            load();
        }

        // 再次判断，如果 Server 创建失败，这里为空，则返回
        if (getServer() == null) {
            log.fatal(sm.getString("catalina.noServer"));
            return;
        }

        long t1 = System.nanoTime();

        // Start the new server
        try {
            // 调用 Server 的 start 方法，启动 Server，默认为 StandardServer#start
            getServer().start();
        } catch (LifecycleException e) {
            log.fatal(sm.getString("catalina.serverStartFail"), e);
            try {
                // 出现异常，则对 Server 进行销毁
                getServer().destroy();
            } catch (LifecycleException e1) {
                log.debug("destroy() failed for failed Server ", e1);
            }
            return;
        }

        long t2 = System.nanoTime();
        if(log.isInfoEnabled()) {
            log.info(sm.getString("catalina.startup", Long.valueOf((t2 - t1) / 1000000)));
        }

        // Register shutdown hook
        // 注册一个关闭钩子
        if (useShutdownHook) {
            if (shutdownHook == null) {
                // 调用 Server 的 stop 方法，释放所有资源
                shutdownHook = new CatalinaShutdownHook();
            }
            Runtime.getRuntime().addShutdownHook(shutdownHook);

            // If JULI is being used, disable JULI's shutdown hook since
            // shutdown hooks run in parallel and log messages may be lost
            // if JULI's hook completes before the CatalinaShutdownHook()
            LogManager logManager = LogManager.getLogManager();
            if (logManager instanceof ClassLoaderLogManager) {
                ((ClassLoaderLogManager) logManager).setUseShutdownHook(
                        false);
            }
        }

        // 使用 await 方法监听停止请求
        if (await) {
            await();
            stop();
        }
    }
```
