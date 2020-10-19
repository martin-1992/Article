
## LifecycleMBeanBase
　　继承 LifecycleBase，initInternal 方法为创建 MBeanServer。

```java
public abstract class LifecycleMBeanBase extends LifecycleBase
        implements JmxEnabled {
    
    // ...
    

    /**
     * 创建 MBeanServer
     */
    @Override
    protected void initInternal() throws LifecycleException {

        // If oname is not null then registration has already happened via
        // preRegister().
        // oname 为空，注册 MBeanServer
        if (oname == null) {
            mserver = Registry.getRegistry(null, null).getMBeanServer();

            oname = register(this, getObjectNameKeyProperties());
        }
    }
}
```
