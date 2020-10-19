
### StandardServer#initInternal

- 调用父类 LifecycleMBeanBase 的 initInternal() 方法，创建 MBeanServer；
- 注册 utilityExecutor、StringCache 和 MBeanFactory；
- globalNamingResources 初始化；
- 获取父类加载器，从上往下，遍历各类加载器。如果该加载器属于 URLClassLoader，则找到结尾为 .jar 的文件，获取 jar 文件的 Manifest 对象，将其添加到 containerManifestResources 中；
- 一个 server 可有多个 services，对每个 services 进行初始化。init() 方法中的抽象方法 initInternal() 默认由子类 [StandardService#initInternal()](https://github.com/martin-1992/Tomcat-Notes/blob/master/Tomcat%20%E7%9A%84%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B/StandardService%23initInternal.md) 实现。

```java
    private Service services[] = new Service[0];

    @Override
    protected void initInternal() throws LifecycleException {
        // 调用父类 LifecycleMBeanBase 的 initInternal() 方法，创建 MBeanServer
        super.initInternal();

        // Initialize utility executor
        reconfigureUtilityExecutor(getUtilityThreadsInternal(utilityThreads));
        register(utilityExecutor, "type=UtilityExecutor");

        // 缓存是全局的，如果 JVM 中存在多个 Server，则同样一个缓存会注册到多个 Server 中，即多个
        // Server 调用的是同一个缓存
        onameStringCache = register(new StringCache(), "type=StringCache");

        // Register the MBeanFactory
        MBeanFactory factory = new MBeanFactory();
        factory.setContainer(this);
        onameMBeanFactory = register(factory, "type=MBeanFactory");

        // Register the naming resources
        // globalNamingResources 初始化
        globalNamingResources.init();

        // Populate the extension validator with JARs from common and shared
        // class loaders
        // 在 Catalina#load() 方法中，设置 getServer().setCatalina(this)，
        if (getCatalina() != null) {
            // 获取父类加载器，从上往下，遍历各类加载器
            ClassLoader cl = getCatalina().getParentClassLoader();
            // Walk the class loader hierarchy. Stop at the system class loader.
            // This will add the shared (if present) and common class loaders
            while (cl != null && cl != ClassLoader.getSystemClassLoader()) {
                // 如果该加载器属于 URLClassLoader，则找到结尾为 .jar 的文件
                if (cl instanceof URLClassLoader) {
                    URL[] urls = ((URLClassLoader) cl).getURLs();
                    for (URL url : urls) {
                        if (url.getProtocol().equals("file")) {
                            try {
                                File f = new File (url.toURI());
                                if (f.isFile() &&
                                        f.getName().endsWith(".jar")) {
                                    // 获取 jar 文件的 Manifest 对象，将其添加到 containerManifestResources 中
                                    ExtensionValidator.addSystemResource(f);
                                }
                            } catch (URISyntaxException e) {
                                // Ignore
                            } catch (IOException e) {
                                // Ignore
                            }
                        }
                    }
                }
                cl = cl.getParent();
            }
        }
        // Initialize our defined Services
        // 一个 server 可有多个 services，对每个 services 进行初始化
        // init() 方法中的抽象方法 initInternal() 默认由子类实现
        // StandardService
        for (int i = 0; i < services.length; i++) {
            services[i].init();
        }
    }
```
