
### Catalina#load
　　创建用于解析 XML 文件的 digester，使用反射为 Catalina 创建 server 组件实例和各种子组件实例。

- 通过变量 loaded，判断是否已经创建 Server，是则返回；
- createStartDigester()，创建用于解析 XML 文件的 digester；
- 获取 server.xml，使用 digester 解析 xml 文件，并设置 Catalina.setServer；
- getServer().init()，因为 Server 继承了 LifecycleBase，这里会调用 [LifecycleBase](https://github.com/martin-1992/Tomcat-Notes/tree/master/LifeCycle) 的 [init 方法](https://github.com/martin-1992/Tomcat-Notes/blob/master/LifeCycle/LifecycleBase.md)。在 init() 方法中有抽象方法 initInternal()，由子类实现的，这里是 StandardServer 实现，所以会调用 [StandardServer#initInternal()](https://github.com/martin-1992/Tomcat-Notes/blob/master/Tomcat%20%E7%9A%84%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B/StandardServer%23initInternal.md) 方法。每个组件都会继承 LifecycleBase；
- 初始化完成，记录耗时时间。

```java
    public void load() {
        // 已经创建了，则返回。一个 Tomcat 实例一个 server 实例
        if (loaded) {
            return;
        }
        loaded = true;

        long t1 = System.nanoTime();

        initDirs();

        // Before digester - it may be needed
        // 在创建 XML 解析器 digester 前，是否要重命名
        initNaming();

        // Set configuration source
        // 设置配置源
        ConfigFileLoader.setSource(new CatalinaBaseConfigurationSource(Bootstrap.getCatalinaBaseFile(), getConfigFile()));
        File file = configFile();

        // Create and execute our Digester
        // 创建用于解析 XML 文件的 digester
        Digester digester = createStartDigester();

        // 获取 server.xml
        try (ConfigurationSource.Resource resource = ConfigFileLoader.getSource().getServerXml()) {
            InputStream inputStream = resource.getInputStream();
            InputSource inputSource = new InputSource(resource.getURI().toURL().toString());
            inputSource.setByteStream(inputStream);
            // 将 Catalina 放入栈顶
            digester.push(this);
            // 使用 digester 解析 xml 文件，因为栈顶为 Catalina，所以这里调用 Catalina.setServer(Sever Object)，
            // 使用反射方法创建 Catalina 使用的 Server 组件和各种子组件如 service、connector、
            // globalNamingResources、service、listener 等
            digester.parse(inputSource);
        } catch (Exception e) {
            if  (file == null) {
                log.warn(sm.getString("catalina.configFail", getConfigFile() + "] or [server-embed.xml"), e);
            } else {
                log.warn(sm.getString("catalina.configFail", file.getAbsolutePath()), e);
                if (file.exists() && !file.canRead()) {
                    log.warn(sm.getString("catalina.incorrectPermissions"));
                }
            }
            return;
        }

        getServer().setCatalina(this);
        getServer().setCatalinaHome(Bootstrap.getCatalinaHomeFile());
        getServer().setCatalinaBase(Bootstrap.getCatalinaBaseFile());

        // Stream redirection
        initStreams();

        // Start the new server
        try {
            // 调用 LifcycleBase 的 init 方法，状态由 NEW -> START，同时会遍历监听器，
            // 监听到该状态的事件，会调用该监听器进行相关逻辑处理
            getServer().init();
        } catch (LifecycleException e) {
            if (Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE")) {
                throw new java.lang.Error(e);
            } else {
                log.error(sm.getString("catalina.initError"), e);
            }
        }

        long t2 = System.nanoTime();
        if(log.isInfoEnabled()) {
            // 初始化完成，耗时
            log.info(sm.getString("catalina.init", Long.valueOf((t2 - t1) / 1000000)));
        }
    }
```

### Catalina#createStartDigester
　　创建用于解析 XML 文件的 digester。

```java
    protected Digester createStartDigester() {
        long t1=System.currentTimeMillis();
        // Initialize the digester
        Digester digester = new Digester();
        digester.setValidating(false);
        digester.setRulesValidation(true);
        Map<Class<?>, List<String>> fakeAttributes = new HashMap<>();
        // Ignore className on all elements
        List<String> objectAttrs = new ArrayList<>();
        objectAttrs.add("className");
        fakeAttributes.put(Object.class, objectAttrs);
        // Ignore attribute added by Eclipse for its internal tracking
        List<String> contextAttrs = new ArrayList<>();
        contextAttrs.add("source");
        fakeAttributes.put(StandardContext.class, contextAttrs);
        // Ignore Connector attribute used internally but set on Server
        List<String> connectorAttrs = new ArrayList<>();
        connectorAttrs.add("portOffset");
        fakeAttributes.put(Connector.class, connectorAttrs);
        digester.setFakeAttributes(fakeAttributes);
        digester.setUseContextClassLoader(true);

        // Configure the actions we will be using
        // 解析到 Server 就创建 StandardServer 实例
        digester.addObjectCreate("Server",
                                 "org.apache.catalina.core.StandardServer",
                                 "className");
        digester.addSetProperties("Server");
        // 调用栈顶的对象的 setter 方法，在 load() 中传入的是 Catalina，所以这里调用的
        // 是 Catalina.setServer()，设置 Server
        digester.addSetNext("Server",
                            "setServer",
                            "org.apache.catalina.Server");

        // 创建 GlobalNamingResources
        digester.addObjectCreate("Server/GlobalNamingResources",
                                 "org.apache.catalina.deploy.NamingResourcesImpl");
        digester.addSetProperties("Server/GlobalNamingResources");
        digester.addSetNext("Server/GlobalNamingResources",
                            "setGlobalNamingResources",
                            "org.apache.catalina.deploy.NamingResourcesImpl");

        // 创建 Listener
        digester.addObjectCreate("Server/Listener",
                                 null, // MUST be specified in the element
                                 "className");
        digester.addSetProperties("Server/Listener");
        digester.addSetNext("Server/Listener",
                            "addLifecycleListener",
                            "org.apache.catalina.LifecycleListener");

        // 创建 Service，默认为 StandardService
        digester.addObjectCreate("Server/Service",
                                 "org.apache.catalina.core.StandardService",
                                 "className");
        digester.addSetProperties("Server/Service");
        digester.addSetNext("Server/Service",
                            "addService",
                            "org.apache.catalina.Service");

        // 创建 Service 下的 Listener
        digester.addObjectCreate("Server/Service/Listener",
                                 null, // MUST be specified in the element
                                 "className");
        digester.addSetProperties("Server/Service/Listener");
        digester.addSetNext("Server/Service/Listener",
                            "addLifecycleListener",
                            "org.apache.catalina.LifecycleListener");

        // 创建 Executor，默认为 StandardThreadExecutor
        digester.addObjectCreate("Server/Service/Executor",
                         "org.apache.catalina.core.StandardThreadExecutor",
                         "className");
        digester.addSetProperties("Server/Service/Executor");

        digester.addSetNext("Server/Service/Executor",
                            "addExecutor",
                            "org.apache.catalina.Executor");

        // 创建 Connector
        digester.addRule("Server/Service/Connector",
                         new ConnectorCreateRule());
        digester.addRule("Server/Service/Connector", new SetAllPropertiesRule(
                new String[]{"executor", "sslImplementationName", "protocol"}));
        digester.addSetNext("Server/Service/Connector",
                            "addConnector",
                            "org.apache.catalina.connector.Connector");

        // ...

        long t2=System.currentTimeMillis();
        if (log.isDebugEnabled()) {
            // 创建 Digester，耗时
            log.debug("Digester for server.xml created " + ( t2-t1 ));
        }
        return digester;

    }
```
