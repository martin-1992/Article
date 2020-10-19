
### StandardService#startInternal
　　先触发监听该 STARTING 状态事件的监听器，然后调用各组件的 start 方法。从代码中可看出一个 Server 实例只有一个 engine，其余为多个，使用同步来调用。

```java
    protected final ArrayList<Executor> executors = new ArrayList<>();
    
    protected Connector connectors[] = new Connector[0];
    private final Object connectorsLock = new Object();

    @Override
    protected void startInternal() throws LifecycleException {

        if(log.isInfoEnabled())
            log.info(sm.getString("standardService.start.name", this.name));
        setState(LifecycleState.STARTING);

        // Start our defined Container first
        if (engine != null) {
            synchronized (engine) {
                engine.start();
            }
        }

        synchronized (executors) {
            for (Executor executor: executors) {
                executor.start();
            }
        }

        mapperListener.start();

        // Start our defined Connectors second
        synchronized (connectorsLock) {
            for (Connector connector: connectors) {
                // If it has already failed, don't try and start it
                if (connector.getState() != LifecycleState.FAILED) {
                    connector.start();
                }
            }
        }
    }
```
