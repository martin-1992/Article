
　　生命周期状态，为枚举类，定义 Tomcat 所有组件的生命周期状态以及对应的事件。比如 INITIALIZING 状态对应的事件为 BEFORE_INIT_EVENT，用于状态转变时，监听器可获取该状态的事件，如果监听器监听该事件，则调用该监听的方法。<br />
　　举例，有些监听器会在初始化前 BEFORE_INIT_EVENT，做一些参数配置等。则当调用初始化方法 init()时，当状态转换到 INITIALIZING 时，会调用监听到 BEFORE_INIT_EVENT 事件的监听器。

```java
public enum LifecycleState {
    NEW(false, null),
    INITIALIZING(false, Lifecycle.BEFORE_INIT_EVENT),
    INITIALIZED(false, Lifecycle.AFTER_INIT_EVENT),
    STARTING_PREP(false, Lifecycle.BEFORE_START_EVENT),
    STARTING(true, Lifecycle.START_EVENT),
    STARTED(true, Lifecycle.AFTER_START_EVENT),
    STOPPING_PREP(true, Lifecycle.BEFORE_STOP_EVENT),
    STOPPING(false, Lifecycle.STOP_EVENT),
    STOPPED(false, Lifecycle.AFTER_STOP_EVENT),
    DESTROYING(false, Lifecycle.BEFORE_DESTROY_EVENT),
    DESTROYED(false, Lifecycle.AFTER_DESTROY_EVENT),
    FAILED(false, null);

    private final boolean available;
    private final String lifecycleEvent;

    private LifecycleState(boolean available, String lifecycleEvent) {
        this.available = available;
        this.lifecycleEvent = lifecycleEvent;
    }

    public boolean isAvailable() {
        return available;
    }

    public String getLifecycleEvent() {
        return lifecycleEvent;
    }
}
```
