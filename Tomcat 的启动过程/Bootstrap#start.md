## Bootstrap#start
　　在 Bootstrap.init() 时会创建 Catalina 的实例，如果没有，则会调用 Bootstrap.init() 方法，然后使用反射的方法调用 Catalina.start() 方法。 

```java
    private Object catalinaDaemon = null;

    /**
     * Start the Catalina daemon.
     * @throws Exception Fatal start error
     */
    public void start() throws Exception {
        // 初始化，创建 Catalina 的实例
        if( catalinaDaemon == null ) init();
        // 调用 Catalina 的 start 方法
        Method method = catalinaDaemon.getClass().getMethod("start", (Class [] )null);
        method.invoke(catalinaDaemon, (Object [])null);

    }
```

### Bootstrap#init
　　初始化类加载器，创建 Catalina 的实例，以便调用它的 start 方法。

```java
    private Object catalinaDaemon = null;
    
    ClassLoader commonLoader = null;
    ClassLoader catalinaLoader = null;
    ClassLoader sharedLoader = null;
    
    public void init() throws Exception {
        // 初始化类加载器
        initClassLoaders();
        // 设置当前线程的上下文加载器
        Thread.currentThread().setContextClassLoader(catalinaLoader);

        SecurityClassLoad.securityClassLoad(catalinaLoader);

        // Load our startup class and call its process() method
        if (log.isDebugEnabled())
            log.debug("Loading startup class");
        // 创建 Catalina 的实例
        Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
        Object startupInstance = startupClass.getConstructor().newInstance();

        // Set the shared extensions class loader
        if (log.isDebugEnabled())
            log.debug("Setting startup class properties");
        String methodName = "setParentClassLoader";
        Class<?> paramTypes[] = new Class[1];
        paramTypes[0] = Class.forName("java.lang.ClassLoader");
        Object paramValues[] = new Object[1];
        paramValues[0] = sharedLoader;
        // 使用反射调用
        Method method = startupInstance.getClass().getMethod(methodName, paramTypes);
        method.invoke(startupInstance, paramValues);

        catalinaDaemon = startupInstance;

    }
```
