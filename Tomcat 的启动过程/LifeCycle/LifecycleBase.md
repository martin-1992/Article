
### LifecycleBase#fireLifecycleEvent

- LifecycleBase 为抽象基类，关于生命周期的方法都为抽象方法，比如 initInternal()、startInternal() 都为抽象方法，交由子类实现。这些方法封装在另外一个方法流程中。下面以 [init()](#init) 方法为例，其他生命周期方法 start()、stop()、destroy() 同理，就不分析了;
- 初始化创建一个监听器数组，每当状态转换时，会调用 fireLifecycleEvent 方法，遍历该监听器数组，如果有监听器监听该状态的事件，则调用该监听器的方法，进行相关的处理。


```java
public abstract class LifecycleBase implements Lifecycle {

    private static final Log log = LogFactory.getLog(LifecycleBase.class);

    private static final StringManager sm = StringManager.getManager(LifecycleBase.class);

    /**
     * 创建监听器的数组
     */
    private final List<LifecycleListener> lifecycleListeners = new CopyOnWriteArrayList<>();

    /**
     * 当前组件的状态
     */
    private volatile LifecycleState state = LifecycleState.NEW;

    /**
     * {@inheritDoc}
     */
    @Override
    public void addLifecycleListener(LifecycleListener listener) {
        lifecycleListeners.add(listener);
    }

    /**
     * 将数组转为 Array
     */
    @Override
    public LifecycleListener[] findLifecycleListeners() {
        return lifecycleListeners.toArray(new LifecycleListener[0]);
    }

    /**
     * 移除某个监听器
     */
    @Override
    public void removeLifecycleListener(LifecycleListener listener) {
        lifecycleListeners.remove(listener);
    }

    /**
     * 相关状态改变后，遍历每个监听器，会触发监听器对其状态进行相关的处理逻辑
     */
    protected void fireLifecycleEvent(String type, Object data) {
        // 构造一个 LifecycleEvent
        LifecycleEvent event = new LifecycleEvent(this, type, data);
        for (LifecycleListener listener : lifecycleListeners) {
            listener.lifecycleEvent(event);
        }
    }
```

### LifecycleBase#init()<a id='init'></a>
　　初始化，使用模板方法设计模式，initInternal 为抽象方法，交由子类实现。其他方法会进行生命周期状态的转换，并遍历每个监听器，检查该状态下的事件是否有相关处理。<br />
　　比如生命周期从 NEW -> INITIALIZING，进行初始化中，会遍历每个监听器，如果有监听器在监听该状态 INITIALIZING 对应的事件 BEFORE_INIT_EVENT，则调用它的方法，进行相关处理。


```java
    @Override
    public final synchronized void init() throws LifecycleException {
        // 判断组件的生命周期是否为 NEW
        if (!state.equals(LifecycleState.NEW)) {
            invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
        }

        try {
            // 生命周期从 NEW -> INITIALIZING，进行初始化中，会遍历每个监听器，如果有监听器
            // 在监听该状态 INITIALIZING 对应的事件 BEFORE_INIT_EVENT，则调用它的方法，进行
            // 相关处理
            setStateInternal(LifecycleState.INITIALIZING, null, false);
            // 抽象方法，交由子类实现
            initInternal();
            // 初始化完毕，同样遍历每个监听器，检查是否有监听该状态 AFTER_INIT_EVENT 对应的
            // 事件 AFTER_INIT_EVENT
            setStateInternal(LifecycleState.INITIALIZED, null, false);
        } catch (Throwable t) {
            handleSubClassException(t, "lifecycleBase.initFail", toString());
        }
    }


    /**
     * Sub-classes implement this method to perform any instance initialisation
     * required.
     *
     * @throws LifecycleException If the initialisation fails
     */
    protected abstract void initInternal() throws LifecycleException;
```

### LifecycleBase#setStateInternal()    
　　检查生命周期状态 state 是否合法，并遍历每个监听器，对该状态的生命周期事件进行处理。比如，在 init() 方法中，在将状态从 NEW -> INITIALIZING，这时会触发 BEFORE_INIT_EVENT 事件（INITIALIZING 对应 BEFORE_INIT_EVENT 事件）。然后遍历每个监听器，如果有监听器在监听这个事件，则会调用它的相关处理逻辑。
  
```java  
    private synchronized void setStateInternal(LifecycleState state, Object data, boolean check)
            throws LifecycleException {
         // ...

        // 调用 fireLifecycleEvent，会遍历每个监听器，对该状态的事件进行相应的逻辑处理
        this.state = state;
        String lifecycleEvent = state.getLifecycleEvent();
        if (lifecycleEvent != null) {
            fireLifecycleEvent(lifecycleEvent, data);
        }
    }
```

### LifecycleBase#handleSubClassException()
　　处理子类异常，当从一个状态到另一个状态转换失败抛出异常，则将该状态设为 FAILED，比如在初始化 init() 方法中，从 NEW -> INITIALIZED 失败。

```java
    private void handleSubClassException(Throwable t, String key, Object... args) throws LifecycleException {
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        String msg = sm.getString(key, args);
        if (getThrowOnFailure()) {
            if (!(t instanceof LifecycleException)) {
                t = new LifecycleException(msg, t);
            }
            throw (LifecycleException) t;
        } else {
            log.error(msg, t);
        }
    }
```
