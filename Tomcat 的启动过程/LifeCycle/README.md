
## LifeCycle
　　LifeCycle 接口为生命周期接口，定义了 Tomcat 所有组件的生命周期。LifeCycle 接口的默认实现类为 [LifecycleBase](https://github.com/martin-1992/Tomcat-Notes/blob/master/LifeCycle/LifecycleBase.md)，该抽象基类实现了 LifeCycle 接口的部分方法，这些方法为公共方法，即每个组件子类的实现逻辑都相同，所以放在基类实现。<br />
　　各组件子类通过继承抽象基类 LifecycleBase，实现其它抽象方法，定制自己的处理逻辑。LifeCycle 和 LifecycleBase，用到了设计模式中的模板方法模式。<br />
　　[LifecycleState](https://github.com/martin-1992/Tomcat-Notes/blob/master/LifeCycle/LifecycleState.md) 为枚举类，用于监听器监听不同状态的事件，做相关的处理。

### LifeCycle 接口　
　　LifeCycle 接口分为三部分：
  
- 第一部分定义各种生命周期的状态；
- 第二部分定义有关生命周期的方法，包含初始化 init()、启动 start()、停止 stop() 和注销 destroy()，每个具体组件都会实现这些方法；
- 第三部分是关于监听器的操作，包含添加、查找和删除。将生命周期的状态看做一个个事件，使用监听器来监听这些事件，并进行逻辑处理（观察者模式）。

```java
public interface Lifecycle {

    // ----------------------------------------------------- Manifest Constants

    /**
     * 定义各种生命周期状态
     */
    public static final String BEFORE_INIT_EVENT = "before_init";

    public static final String AFTER_INIT_EVENT = "after_init";

    public static final String START_EVENT = "start";

    public static final String BEFORE_START_EVENT = "before_start";

    public static final String AFTER_START_EVENT = "after_start";

    public static final String STOP_EVENT = "stop";

    public static final String BEFORE_STOP_EVENT = "before_stop";

    public static final String AFTER_STOP_EVENT = "after_stop";

    public static final String AFTER_DESTROY_EVENT = "after_destroy";

    public static final String BEFORE_DESTROY_EVENT = "before_destroy";

    public static final String PERIODIC_EVENT = "periodic";

    public static final String CONFIGURE_START_EVENT = "configure_start";

    public static final String CONFIGURE_STOP_EVENT = "configure_stop";


    // --------------------------------------------------------- Public Methods


    /**
     * 对监听器的操作，包括添加、查找和删除，监听器是存在数组中
     */
    public void addLifecycleListener(LifecycleListener listener);

    public LifecycleListener[] findLifecycleListeners();

    public void removeLifecycleListener(LifecycleListener listener);


    /**
     * 初始化方法
     */
    public void init() throws LifecycleException;

    /**
     * 启动方法
     */
    public void start() throws LifecycleException;


    /**
     * 停止方法
     */
    public void stop() throws LifecycleException;

    /**
     * 注销，释放资源方法
     */
    public void destroy() throws LifecycleException;


    /**
     * 获取组件的生命周期状态
     */
    public LifecycleState getState();


    /**
     * 返回当前组件的状态名
     */
    public String getStateName();


    /**
     * Marker interface used to indicate that the instance should only be used
     * once. Calling {@link #stop()} on an instance that supports this interface
     * will automatically call {@link #destroy()} after {@link #stop()}
     * completes.
     */
    public interface SingleUse {
    }
}
```
