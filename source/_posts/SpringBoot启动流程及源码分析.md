---
title: SpringApplication源码解析
description: SpringApplication源码解析,记录自己的理解
date: 2018-03-31 13:59:00
comments: true
tags: 
    - Spring  
    - SpringBoot
categories:
    - Spring
    - SpringBoot
---
 
 SpringBoot启动流程源码解析，版本号为1.5.7

# SpringApplication创建

```java
run方法调用
public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
        return (new SpringApplication(sources)).run(args);
}

构造方法
public SpringApplication(Object... sources) {
        this.bannerMode = Mode.CONSOLE;
        this.logStartupInfo = true;
        this.addCommandLineProperties = true;
        this.headless = true;
        this.registerShutdownHook = true;
        this.additionalProfiles = new HashSet();
        this.initialize(sources);
}
实例化对象
 private void initialize(Object[] sources) {
        if (sources != null && sources.length > 0) {
            this.sources.addAll(Arrays.asList(sources));
        }

        this.webEnvironment = this.deduceWebEnvironment();  //判断是否为Web环境 根据是否存在"javax.servlet.Servlet", "org.springframework.web.context.ConfigurableWebApplicationContext"这两个类判断 如果都存在，说明是web环境
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class)); //加载Initializer
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class)); // 加载Listeners
        this.mainApplicationClass = this.deduceMainApplicationClass(); //推断main方法所在类
}

```
## 加载Initializers
```java
private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type) {
    return this.getSpringFactoriesInstances(type, new Class[0]);
}

private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
    Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));  //通过spring.factories返回名字，并通过createSpringFactoriesInstances创建对象
    List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```
以上关键代码为SpringFactoriesLoader.loadFactoryNames(type, classLoader) 源码如下,通过以下代码返回spring.factories中ApplicationContextInitializer配置的value返回需要创建对象的名称
```java
    
    public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
        String factoryClassName = factoryClass.getName();

        try {
            Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
            ArrayList result = new ArrayList();

            while(urls.hasMoreElements()) {
                URL url = (URL)urls.nextElement();
                Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
                String factoryClassNames = properties.getProperty(factoryClassName);
                result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
            }

            return result;
        } catch (IOException var8) {
            throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() + "] factories from location [" + "META-INF/spring.factories" + "]", var8);
        }
    }
```
通过以上代码分析得知通过spring.factories文件加载所有Initializers，spring.factories文件如下所示
```properties
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.context.web.ServerPortInfoApplicationContextInitializer
```


通过类全称创建对象，对应上一步会生成ConfigurationWarningsApplicationContextInitializer，ContextIdApplicationContextInitializer，DelegatingApplicationContextInitializer，ServerPortInfoApplicationContextInitializer四个对象
```java
private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args, Set<String> names) {
    List<T> instances = new ArrayList(names.size());
    Iterator var7 = names.iterator();

    while(var7.hasNext()) {
        String name = (String)var7.next();

        try {
            Class<?> instanceClass = ClassUtils.forName(name, classLoader);
            Assert.isAssignable(type, instanceClass);
            Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
            T instance = BeanUtils.instantiateClass(constructor, args);
            instances.add(instance);
        } catch (Throwable var12) {
            throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, var12);
        }
    }

    return instances;
}
```
各个Initializer作用
![initializer][initializer]

## 创建listeners

listeners与initiazer类似，只不过类型改为ApplicationListener,spring.factories文件中值以下代码所示，返回结果为
```properties
org.springframework.context.ApplicationListener=\
org.springframework.boot.builder.ParentContextCloserApplicationListener,\
org.springframework.boot.context.FileEncodingApplicationListener,\
org.springframework.boot.context.config.AnsiOutputApplicationListener,\
org.springframework.boot.context.config.ConfigFileApplicationListener,\
org.springframework.boot.context.config.DelegatingApplicationListener,\
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener,\
org.springframework.boot.logging.ClasspathLoggingApplicationListener,\
org.springframework.boot.logging.LoggingApplicationListener
```
以上Listener作用
![Listener][Listener]

 推断main方法所在类，通过获取当前调用栈，找到入口方法main所在的类，并将其复制给SpringApplication对象的成员变量mainApplicationClass。

```java
    private Class<?> deduceMainApplicationClass() {
        try {
            StackTraceElement[] stackTrace = (new RuntimeException()).getStackTrace();
            StackTraceElement[] var2 = stackTrace;
            int var3 = stackTrace.length;

            for(int var4 = 0; var4 < var3; ++var4) {
                StackTraceElement stackTraceElement = var2[var4];
                if ("main".equals(stackTraceElement.getMethodName())) {
                    return Class.forName(stackTraceElement.getClassName());
                }
            }
        } catch (ClassNotFoundException var6) {
            ;
        }

        return null;
    }

```

# SpringApplication run()方法

```java
    public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();  //记录时间
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        FailureAnalyzers analyzers = null;
        this.configureHeadlessProperty();
        SpringApplicationRunListeners listeners = this.getRunListeners(args); //加载Listeners
        listeners.starting(); //listeners启动 这一步发布ApplicationStartedEvent

        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);  //context创建前环境配置
            Banner printedBanner = this.printBanner(environment); //打印banner
            context = this.createApplicationContext(); //创建上下文
            new FailureAnalyzers(context);//异常分析
            this.prepareContext(context, environment, listeners, applicationArguments, printedBanner); //context前置处理
            this.refreshContext(context); //context刷新
            this.afterRefresh(context, applicationArguments); //后置处理
            listeners.finished(context, (Throwable)null); //listener完成
            stopWatch.stop();
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }

            return context;
        } catch (Throwable var9) {
            this.handleRunFailure(context, listeners, (FailureAnalyzers)analyzers, var9);
            throw new IllegalStateException(var9);
        }
    }
```

## 加载Listeners
```java
    private SpringApplicationRunListeners getRunListeners(String[] args) {
        Class<?>[] types = new Class[]{SpringApplication.class, String[].class};
        // 传入SpringApplicationRunListener 
        return new SpringApplicationRunListeners(logger, this.getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
    }
```
与以上加载initialazer类似，类名改为SpringApplicationRunListener，springfactories配置

```properties
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```

### EventPublishingRunListener作用

```java
    //EventPublishingRunListener 类方法,该类实现SpringApplicationRunListener 接口

public interface SpringApplicationRunListener {
    void starting(); //在run方法业务逻辑执行、应用上下文初始化前调用此方法

    void environmentPrepared(ConfigurableEnvironment var1); //当环境准备完成，应用上下文被创建之前调用此方法 读取配置文件在这个步骤进行

    void contextPrepared(ConfigurableApplicationContext var1); //在应用上下文被创建和准备完成之后，但上下文相关代码被加载执行之前调用。
    //因为上下文准备事件和上下文加载事件难以明确区分，所以这个方法一般没有具体实现。

    void contextLoaded(ConfigurableApplicationContext var1); //当上下文加载完成之后，自定义bean完全加载完成之前调用此方法。

    void finished(ConfigurableApplicationContext var1, Throwable var2); //当run方法执行完成，或执行过程中发现异常时调用此方法。
}

    //EventPublishingRunListener构造方法，将SpringApplication中的**所有**ApplicationListener放入ApplicationEventMulticaster广播器
    public EventPublishingRunListener(SpringApplication application, String[] args) {
        this.application = application;
        this.args = args;
        this.initialMulticaster = new SimpleApplicationEventMulticaster();
        Iterator var3 = application.getListeners().iterator();

        while(var3.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var3.next();
            this.initialMulticaster.addApplicationListener(listener);
        }
    }

    //开始方法
    public void starting() {
        this.initialMulticaster.multicastEvent(new ApplicationStartedEvent(this.application, this.args)); //ApplicationStartedEvent这个事件
    }

    //SimpleApplicationEventMulticaster方法
    public void multicastEvent(ApplicationEvent event) {
        this.multicastEvent(event, this.resolveDefaultEventType(event));
    }

    public void multicastEvent(final ApplicationEvent event, ResolvableType eventType) {
        ResolvableType type = eventType != null ? eventType : this.resolveDefaultEventType(event);
        Iterator var4 = this.getApplicationListeners(event, type).iterator();

        while(var4.hasNext()) {
            final ApplicationListener<?> listener = (ApplicationListener)var4.next();
            Executor executor = this.getTaskExecutor();
            if (executor != null) {
                executor.execute(new Runnable() {
                    public void run() {
                        SimpleApplicationEventMulticaster.this.invokeListener(listener, event);
                    }
                });
            } else {
                this.invokeListener(listener, event);
            }
        }
    }

    protected void invokeListener(ApplicationListener<?> listener, ApplicationEvent event) {
        ErrorHandler errorHandler = this.getErrorHandler();
        if (errorHandler != null) {
            try {
                this.doInvokeListener(listener, event);
            } catch (Throwable var5) {
                errorHandler.handleError(var5);
            }
        } else {
            this.doInvokeListener(listener, event);
        }

    }

    private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
        try {
            listener.onApplicationEvent(event); // 最后调用listener这个方法，这个方法可拓展，对应之前SpringApplication成员属性listeners
        } catch (ClassCastException var6) {
            String msg = var6.getMessage();
            if (msg != null && !msg.startsWith(event.getClass().getName())) {
                throw var6;
            }

            Log logger = LogFactory.getLog(this.getClass());
            if (logger.isDebugEnabled()) {
                logger.debug("Non-matching event type for listener: " + listener, var6);
            }
        }

    }
```

### listener.onApplicationEvent 方法与SpringApplication成员属性对应

```java
    // ParentContextCloserApplicationListener 
    public void onApplicationEvent(ParentContextAvailableEvent event) {
        this.maybeInstallListenerInParent(event.getApplicationContext());
    }
    //FileEncodingApplicationListener
    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
        RelaxedPropertyResolver resolver = new RelaxedPropertyResolver(event.getEnvironment(), "spring.");
        if (resolver.containsProperty("mandatoryFileEncoding")) {
            String encoding = System.getProperty("file.encoding");
            String desired = resolver.getProperty("mandatoryFileEncoding");
            if (encoding != null && !desired.equalsIgnoreCase(encoding)) {
                logger.error("System property 'file.encoding' is currently '" + encoding + "'. It should be '" + desired + "' (as defined in 'spring.mandatoryFileEncoding').");
                logger.error("Environment variable LANG is '" + System.getenv("LANG") + "'. You could use a locale setting that matches encoding='" + desired + "'.");
                logger.error("Environment variable LC_ALL is '" + System.getenv("LC_ALL") + "'. You could use a locale setting that matches encoding='" + desired + "'.");
                throw new IllegalStateException("The Java Virtual Machine has not been configured to use the desired default character encoding (" + desired + ").");
            }
        }

    }

    //AnsiOutputApplicationListener
    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
        RelaxedPropertyResolver resolver = new RelaxedPropertyResolver(event.getEnvironment(), "spring.output.ansi.");
        if (resolver.containsProperty("enabled")) {
            String enabled = resolver.getProperty("enabled");
            AnsiOutput.setEnabled((Enabled)Enum.valueOf(Enabled.class, enabled.toUpperCase()));
        }

        if (resolver.containsProperty("console-available")) {
            AnsiOutput.setConsoleAvailable((Boolean)resolver.getProperty("console-available", Boolean.class));
        }

    }
    //ConfigFileApplicationListener ApplicationEnvironmentPreparedEvent
    public void onApplicationEvent(ApplicationEvent event) {
        if (event instanceof ApplicationEnvironmentPreparedEvent) {
            this.onApplicationEnvironmentPreparedEvent((ApplicationEnvironmentPreparedEvent)event);
        }

        if (event instanceof ApplicationPreparedEvent) {
            this.onApplicationPreparedEvent(event);
        }
    }

    //DelegatingApplicationListener 监控ApplicationEnvironmentPreparedEvent

    public void onApplicationEvent(ApplicationEvent event) {
        if (event instanceof ApplicationEnvironmentPreparedEvent) {
            List<ApplicationListener<ApplicationEvent>> delegates = this.getListeners(((ApplicationEnvironmentPreparedEvent)event).getEnvironment());
            if (delegates.isEmpty()) {
                return;
            }

            this.multicaster = new SimpleApplicationEventMulticaster();
            Iterator var3 = delegates.iterator();

            while(var3.hasNext()) {
                ApplicationListener<ApplicationEvent> listener = (ApplicationListener)var3.next();
                this.multicaster.addApplicationListener(listener);
            }
        }

        if (this.multicaster != null) {
            this.multicaster.multicastEvent(event);
        }

    }

    //LiquibaseServiceLocatorApplicationListener  这个与上面创建的事件对应，都是ApplicationStartingEvent，所以会执行
    public void onApplicationEvent(ApplicationStartingEvent event) {
        if (ClassUtils.isPresent("liquibase.servicelocator.ServiceLocator", (ClassLoader)null)) {
            (new LiquibaseServiceLocatorApplicationListener.LiquibasePresent()).replaceServiceLocator();
        }

    }
   //其他Listener省略
```

###  自定义ApplicationListener，用于初始化后执行特定方法原理
创建JavaBean实现ApplicationListener<E extends ApplicationEvent>  在容器启动时，坚持容器中所有ApplicationListener并将注册到SimpleApplicationEventMulticaster集合中，在 SimpleApplicationEventMulticaster 中，可以看到 multicastEvent 方法中遍历了所有的 Listener，并依次调用 Listener 的 onApplicationEvent 方法发布事件。

## prepareEnvironment方法

在Context创建前准备环境
```java
   //SpringApplication 
    private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments) {
        ConfigurableEnvironment environment = this.getOrCreateEnvironment();
        this.configureEnvironment((ConfigurableEnvironment)environment, applicationArguments.getSourceArgs());
        listeners.environmentPrepared((ConfigurableEnvironment)environment);  //注意 配置
        // 此时发布ApplicationEnvironmentPreparedEvent事件，这一步读取配置文件
        if (!this.webEnvironment) {
            environment = (new EnvironmentConverter(this.getClassLoader())).convertToStandardEnvironmentIfNecessary((ConfigurableEnvironment)environment);
        }

        return (ConfigurableEnvironment)environment;
    }

    protected void configureEnvironment(ConfigurableEnvironment environment, String[] args) {
        //配置属性源
        this.configurePropertySources(environment, args);
        //配置profile
        this.configureProfiles(environment, args);
    }

    protected void configurePropertySources(ConfigurableEnvironment environment, String[] args) {
        MutablePropertySources sources = environment.getPropertySources();
        if (this.defaultProperties != null && !this.defaultProperties.isEmpty()) {
            sources.addLast(new MapPropertySource("defaultProperties", this.defaultProperties));
        }

        if (this.addCommandLineProperties && args.length > 0) {
            String name = "commandLineArgs";
            if (sources.contains(name)) {
                PropertySource<?> source = sources.get(name);
                CompositePropertySource composite = new CompositePropertySource(name);
                composite.addPropertySource(new SimpleCommandLinePropertySource(name + "-" + args.hashCode(), args));
                composite.addPropertySource(source);
                sources.replace(name, composite);
            } else {
                sources.addFirst(new SimpleCommandLinePropertySource(args));
            }
        }

    }


    protected void configureProfiles(ConfigurableEnvironment environment, String[] args) {
        environment.getActiveProfiles();
        Set<String> profiles = new LinkedHashSet(this.additionalProfiles);
        profiles.addAll(Arrays.asList(environment.getActiveProfiles()));
        environment.setActiveProfiles((String[])profiles.toArray(new String[profiles.size()]));
    }

    // AbstractEnvironment 
    public String[] getActiveProfiles() {
        return StringUtils.toStringArray(this.doGetActiveProfiles());
    }

    protected Set<String> doGetActiveProfiles() {
        Set var1 = this.activeProfiles;
        synchronized(this.activeProfiles) {
            if (this.activeProfiles.isEmpty()) {
                String profiles = this.getProperty("spring.profiles.active");  //系统配置 通过配置决定使用哪个配置文件
                if (StringUtils.hasText(profiles)) {
                    this.setActiveProfiles(StringUtils.commaDelimitedListToStringArray(StringUtils.trimAllWhitespace(profiles)));
                }
            }

            return this.activeProfiles;
        }
    }
```

###  listeners.environmentPrepared((ConfigurableEnvironment)environment)方法
```java
    //SpringApplicationRunListeners
    public void environmentPrepared(ConfigurableEnvironment environment) {
        Iterator var2 = this.listeners.iterator();

        while(var2.hasNext()) {
            SpringApplicationRunListener listener = (SpringApplicationRunListener)var2.next();
            listener.environmentPrepared(environment);
        }

    }

    //EventPublishingRunListener
    public void environmentPrepared(ConfigurableEnvironment environment) {
        this.initialMulticaster.multicastEvent(new ApplicationEnvironmentPreparedEvent(this.application, this.args, environment));//创建事件类型为ApplicationEnvironmentPreparedEvent
    }
```


上面代码创建ApplicationEnvironmentPreparedEvent，ConfigFileApplicationListener监听响应这个事件
```java
    //ConfigFileApplicationListener
    public void onApplicationEvent(ApplicationEvent event) {
        if (event instanceof ApplicationEnvironmentPreparedEvent) {
            this.onApplicationEnvironmentPreparedEvent((ApplicationEnvironmentPreparedEvent)event);
        }

        if (event instanceof ApplicationPreparedEvent) {
            this.onApplicationPreparedEvent(event);
        }

    }



    private void onApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
        List<EnvironmentPostProcessor> postProcessors = this.loadPostProcessors(); //
        postProcessors.add(this);
        AnnotationAwareOrderComparator.sort(postProcessors);
        Iterator var3 = postProcessors.iterator();

        while(var3.hasNext()) {
            EnvironmentPostProcessor postProcessor = (EnvironmentPostProcessor)var3.next();
            postProcessor.postProcessEnvironment(event.getEnvironment(), event.getSpringApplication());
        }

    }

    List<EnvironmentPostProcessor> loadPostProcessors() {
        return SpringFactoriesLoader.loadFactories(EnvironmentPostProcessor.class, this.getClass().getClassLoader());
    }
```
加载EnvironmentPostProcessor 
```properties
# Environment Post Processors
org.springframework.boot.env.EnvironmentPostProcessor=\
org.springframework.boot.cloud.CloudFoundryVcapEnvironmentPostProcessor,\
org.springframework.boot.env.SpringApplicationJsonEnvironmentPostProcessor

```

LoggingApplicationListener同样响应这个事件，对LogbackLoggingSystem初始化工作，具体未知
```java
    // LoggingApplicationListener  加载log日志打印，具体未知
    private void onApplicationStartingEvent(ApplicationStartingEvent event) {
        this.loggingSystem = LoggingSystem.get(event.getSpringApplication().getClassLoader());
        this.loggingSystem.beforeInitialize();
    }

    private void onApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
        if (this.loggingSystem == null) {
            this.loggingSystem = LoggingSystem.get(event.getSpringApplication().getClassLoader());
        }

        this.initialize(event.getEnvironment(), event.getSpringApplication().getClassLoader());
    }

    protected void initialize(ConfigurableEnvironment environment, ClassLoader classLoader) {
        (new LoggingSystemProperties(environment)).apply();
        LogFile logFile = LogFile.get(environment);
        if (logFile != null) {
            logFile.applyToSystemProperties();
        }

        this.initializeEarlyLoggingLevel(environment);
        this.initializeSystem(environment, this.loggingSystem, logFile);
        this.initializeFinalLoggingLevels(environment, this.loggingSystem);
        this.registerShutdownHookIfNecessary(environment, this.loggingSystem);
    }
```

## createApplicationContext() 创建上下文,判断是否为web环境，通常为web项目，只分析web环境及返回AnnotationConfigEmbeddedWebApplicationContext
```java

    protected ConfigurableApplicationContext createApplicationContext() {
        Class<?> contextClass = this.applicationContextClass;
        if (contextClass == null) {
            try {
                contextClass = Class.forName(this.webEnvironment ? "org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext" : "org.springframework.context.annotation.AnnotationConfigApplicationContext");
            } catch (ClassNotFoundException var3) {
                throw new IllegalStateException("Unable create a default ApplicationContext, please specify an ApplicationContextClass", var3);
            }
        }

        return (ConfigurableApplicationContext)BeanUtils.instantiate(contextClass);
    }
```


AnnotationConfigEmbeddedWebApplicationContext 类图如下所示
![webapplication][webapplication]

AnnotationConfigEmbeddedWebApplicationContext构造方法如下 ,通过以下代码得知，只是为成员变量赋值
```java
    public AnnotationConfigEmbeddedWebApplicationContext() {
        this.reader = new AnnotatedBeanDefinitionReader(this); 
        this.scanner = new ClassPathBeanDefinitionScanner(this);
    }
```
AnnotatedBeanDefinitionReader 解析并读取BeanDefinition
ClassPathBeanDefinitionScanner 用于扫描classpath目录下带注解的beanclass


## 接着上面prepareContext方法
```java

    private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
        context.setEnvironment(environment); //将环境赋值与context
        this.postProcessApplicationContext(context);
        this.applyInitializers(context); 
        listeners.contextPrepared(context); //调用ApplicationPreparedEvent
        if (this.logStartupInfo) {
            this.logStartupInfo(context.getParent() == null);
            this.logStartupProfileInfo(context);
        }

        context.getBeanFactory().registerSingleton("springApplicationArguments", applicationArguments);
        if (printedBanner != null) {
            context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
        }

        Set<Object> sources = this.getSources();
        Assert.notEmpty(sources, "Sources must not be empty");
        this.load(context, sources.toArray(new Object[sources.size()]));
        listeners.contextLoaded(context);//调用ApplicationPreparedEvent
    }

    // AnnotationConfigEmbeddedWebApplicationContext 环境赋值
    public void setEnvironment(ConfigurableEnvironment environment) {
        super.setEnvironment(environment);
        this.reader.setEnvironment(environment);
        this.scanner.setEnvironment(environment);
    }
```

### listeners.contextPrepared(context)方法
```java
    //SpringApplicationRunListeners
    public void contextPrepared(ConfigurableApplicationContext context) {
        Iterator var2 = this.listeners.iterator();

        while(var2.hasNext()) {
            SpringApplicationRunListener listener = (SpringApplicationRunListener)var2.next();
            listener.contextPrepared(context);
        }
    }
     //EventPublishingRunListener 空方法
    public void contextPrepared(ConfigurableApplicationContext context) {
    }
```

### isteners.contextLoaded(context);

```java
    //SpringApplicationRunListeners
    public void contextLoaded(ConfigurableApplicationContext context) {
        Iterator var2 = this.listeners.iterator();

        while(var2.hasNext()) {
            SpringApplicationRunListener listener = (SpringApplicationRunListener)var2.next();
            listener.contextLoaded(context);
        }
    }

    //EventPublishingRunListener
    public void contextLoaded(ConfigurableApplicationContext context) {
        ApplicationListener listener;
        for(Iterator var2 = this.application.getListeners().iterator(); var2.hasNext(); context.addApplicationListener(listener)) {
            listener = (ApplicationListener)var2.next();
            if (listener instanceof ApplicationContextAware) {
                ((ApplicationContextAware)listener).setApplicationContext(context);
            }
        }

        this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
    }
```


[initializer]:/images/springboot/initializer.png
[Listener]:/images/springboot/SpringBootApplicationListener.png
[webapplication]:/images/springboot/webApplicationContext.jpg