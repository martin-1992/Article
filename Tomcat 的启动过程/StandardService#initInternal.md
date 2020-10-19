### StandardService#initInternal
　　各个组件在 [Catalina#load](https://github.com/martin-1992/Tomcat-Notes/blob/master/Tomcat%20%E7%9A%84%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B/Catalina%23load.md) 使用 Digester 解析 Server.xml 文件，已创建并配置好参数。然后是继承生命周期接口 [LifeCycle](https://github.com/martin-1992/Tomcat-Notes/tree/master/LifeCycle) 调用 init() 方法，会调用具体子类的 initInternal() 方法。

- [engine.init()]()；
- [executor.init()]()；
- [mapperListener.init()]；
- [connector.init()]。

```java
    @Override
    protected void initInternal() throws LifecycleException {
        // 调用父类 LifecycleMBeanBase 的 initInternal() 方法，创建 MBeanServer
        super.initInternal();

        if (engine != null) {
            engine.init();
        }

        // Initialize any Executors
        // 遍历 Executor 数组，对每个 Executor 进行初始化
        for (Executor executor : findExecutors()) {
            if (executor instanceof JmxEnabled) {
                ((JmxEnabled) executor).setDomain(getDomain());
            }
            executor.init();
        }

        // Initialize mapper listener
        mapperListener.init();

        // Initialize our defined Connectors
        synchronized (connectorsLock) {
            for (Connector connector : connectors) {
                connector.init();
            }
        }
    }
```

#### StandardService#findExecutors
　　Executor 组件在 [Catalina#load](https://github.com/martin-1992/Tomcat-Notes/blob/master/Tomcat%20%E7%9A%84%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B/Catalina%23load.md) 使用 Digester 解析 Server.xml 文件，已创建并配置好参数。这里是返回 Executor 数组。

```java
    @Override
    public Executor[] findExecutors() {
        synchronized (executors) {
            Executor[] arr = new Executor[executors.size()];
            executors.toArray(arr);
            return arr;
        }
    }
```
