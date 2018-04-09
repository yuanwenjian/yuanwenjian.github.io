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

# SpringBoot启动源码解析

## SpringApplication创建源码解析
```java
package org.springframework.boot.SpringApplication;

public SpringApplication(ResourceLoader resourceLoader, Object... sources) {
        this.bannerMode = Mode.CONSOLE;
        this.logStartupInfo = true;
        this.addCommandLineProperties = true;
        this.headless = true;
        this.registerShutdownHook = true;
        this.additionalProfiles = new HashSet();
        this.resourceLoader = resourceLoader;
        this.initialize(sources);
    }

    private void initialize(Object[] sources) {
        if(sources != null && sources.length > 0) {
            this.sources.addAll(Arrays.asList(sources));
        }

        this.webEnvironment = this.deduceWebEnvironment(); //判断是否为Web环境 根据"javax.servlet.Servlet", "org.springframework.web.context.ConfigurableWebApplicationContext"判断 对
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class)); //添加Initializer 通过ApplicationContextInitializer获取所有spring.factories文件中key为 org.springframework.context.ApplicationContextInitializer的类
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class)); //添加ApplicationListener过程类似，
        this.mainApplicationClass = this.deduceMainApplicationClass(); //推断main的定义类
    }
```
## run方法
```java
public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        this.configureHeadlessProperty(); //
        SpringApplicationRunListeners listeners = this.getRunListeners(args); //获取所有SpringApplicationRunListener配置的listteners
        listeners.started(); //通知所有的listeners启动

        ...
        context = this.doRun(listeners, args);
        return context;
        ...
    }

//个人认为核心方法
private ConfigurableApplicationContext doRun(SpringApplicationRunListeners listeners, String... args) {
        ConfigurableEnvironment environment = this.getOrCreateEnvironment();
        this.configureEnvironment(environment, args); //两行为配置environment
        listeners.environmentPrepared(environment);
        if(this.isWebEnvironment(environment) && !this.webEnvironment) {
            environment = this.convertToStandardEnvironment(environment);
        }

        if(this.bannerMode != Mode.OFF) {
            this.printBanner(environment); //打印banner
        }

        ConfigurableApplicationContext context = this.createApplicationContext();
        context.setEnvironment(environment);//
        this.postProcessApplicationContext(context);
        this.applyInitializers(context); //对conten进一步处理
        listeners.contextPrepared(context); //8 
        if(this.logStartupInfo) {
            this.logStartupInfo(context.getParent() == null);
            this.logStartupProfileInfo(context);
        }

        DefaultApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        context.getBeanFactory().registerSingleton("springApplicationArguments", applicationArguments);
        Set sources = this.getSources();
        Assert.notEmpty(sources, "Sources must not be empty");
        this.load(context, sources.toArray(new Object[sources.size()]));//
        listeners.contextLoaded(context);
        this.refresh(context);
        if(this.registerShutdownHook) {
            try {
                context.registerShutdownHook();
            } catch (AccessControlException var8) {
                ;
            }
        }

        this.afterRefresh(context, (ApplicationArguments)applicationArguments);
        listeners.finished(context, (Throwable)null);
        return context;
    }

```

## 获取配置的类全称
```java
public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
        String factoryClassName = factoryClass.getName();

        try {
            Enumeration ex = classLoader != null?classLoader.getResources("META-INF/spring.factories"):ClassLoader.getSystemResources("META-INF/spring.factories"); //获取classpath所有的spring.factories文件
            ArrayList result = new ArrayList();

            while(ex.hasMoreElements()) {
                URL url = (URL)ex.nextElement();
                Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
                String factoryClassNames = properties.getProperty(factoryClassName);//根据配置的key获取类型
                result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));//使用，拆分放入集合
            }

            return result;
        } catch (IOException var8) {
            ...
        }
    }
```

## SpringBoot jar包中spring.factories文件中
```
# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.builder.ParentContextCloserApplicationListener,\
org.springframework.boot.context.FileEncodingApplicationListener,\
org.springframework.boot.context.config.AnsiOutputApplicationListener,\
org.springframework.boot.context.config.ConfigFileApplicationListener,\
org.springframework.boot.context.config.DelegatingApplicationListener,\
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener,\
org.springframework.boot.logging.ClasspathLoggingApplicationListener,\
org.springframework.boot.logging.LoggingApplicationListener

# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.context.web.ServerPortInfoApplicationContextInitializer

org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```

## 总结
SpringBoot启动时会扫描classpath下所有的META-INF/spring.factories文件，通过SpringFactoriesLoader.loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader)获取类的名称全称，在通过反射创建配置的类