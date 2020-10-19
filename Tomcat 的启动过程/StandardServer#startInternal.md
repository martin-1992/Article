
### StandardServer#startInternal
　　调用每个 Service 组件的 start 方法。

- fireLifecycleEvent 方法和 setState 为 LifecycleBase 类的方法，当状态改变，从 INIT -> START，触发监听该状态事件的监听器；
- globalNamingResources 在解析 [Catalina#load](https://github.com/martin-1992/Tomcat-Notes/blob/master/Tomcat%20%E7%9A%84%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B/Catalina%23load.md) 解析 server.xml 时已创建；
-  用一个实例对象作为锁，保证线程安全，遍历每个 service，调用 start 方法，每个 service 实现自己的 startInternal() 方法，默认为 [StandardService#startInternal](https://github.com/martin-1992/Tomcat-Notes/blob/master/Tomcat%20%E7%9A%84%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B/StandardService%23startInternal.md)；


```java
    private NamingResourcesImpl globalNamingResources = null;
    
    private final Object servicesLock = new Object();
    
    @Override
    protected void startInternal() throws LifecycleException {
        // fireLifecycleEvent 方法和 setState 为 LifecycleBase 类的方法，
        // 当状态改变，从 INIT -> START，触发监听该状态事件的监听器
        fireLifecycleEvent(CONFIGURE_START_EVENT, null);
        setState(LifecycleState.STARTING);
        // 在解析 server.xml 时已创建
        globalNamingResources.start();

        // Start our defined Services
        // 使用一个实例对象作为锁，保证线程安全，遍历每个 service，调用
        // start 方法，每个 service 实现自己的 startInternal() 方法
        synchronized (servicesLock) {
            for (int i = 0; i < services.length; i++) {
                services[i].start();
            }
        }

        if (periodicEventDelay > 0) {
            monitorFuture = getUtilityExecutor().scheduleWithFixedDelay(
                    new Runnable() {
                        @Override
                        public void run() {
                            startPeriodicLifecycleEvent();
                        }
                    }, 0, 60, TimeUnit.SECONDS);
        }
    }
```
