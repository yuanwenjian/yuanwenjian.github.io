---
title: SpringCloud Config文档
description: Spring Cloud Config为分布式系统中的外部化配置提供服务器和客户端支持
date: 2018-03-26 22:48:00
comments: true
tags: 
    - Spring  
    - SpringBoot
categories:
    - Spring
    - SpringBoot
---
# SpringApplication执行流程
1. 如果我们使用的是SpringApplication的静态run方法，那么，这个方法里面首先要创建一个SpringApplication对象实例，然后调用这个创建好的SpringApplication的实例方法。在SpringApplication实例初始化的时候，它会提前做几件事情：
    - 根据classpath里面是否存在某个特征类（org.springframework.web.context.ConfigurableWebApplicationContext）来决定是否应该创建一个为Web应用使用的ApplicationContext类型。
    - 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationContextInitializer。
    - 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationListener。
推断并设置main方法的定义类。
2. SpringApplication实例初始化完成并且完成设置后，就开始执行run方法的逻辑了，方法执行伊始，首先遍历执行所有通过SpringFactoriesLoader可以查找到并加载的SpringApplicationRunListener。调用它们的started()方法，告诉这些SpringApplicationRunListener，“嘿，SpringBoot应用要开始执行咯！”。
3. 创建并配置当前Spring Boot应用将要使用的Environment（包括配置要使用的PropertySource以及Profile）。
4. 遍历调用所有SpringApplicationRunListener的environmentPrepared()的方法，告诉他们：“当前SpringBoot应用使用的Environment准备好了咯！”。
5. 如果SpringApplication的showBanner属性被设置为true，则打印banner。
6. 根据用户是否明确设置了applicationContextClass类型以及初始化阶段的推断结果，决定该为当前SpringBoot应用创建什么类型的ApplicationContext并创建完成，然后根据条件决定是否添加ShutdownHook，决定是否使用自定义的BeanNameGenerator，决定是否使用自定义的ResourceLoader，当然，最重要的，将之前准备好的Environment设置给创建好的ApplicationContext使用。
7. ApplicationContext创建好之后，SpringApplication会再次借助Spring-FactoriesLoader，查找并加载classpath中所有可用的ApplicationContext-Initializer，然后遍历调用这些ApplicationContextInitializer的initialize（applicationContext）方法来对已经创建好的ApplicationContext进行进一步的处理。

8. 遍历调用所有SpringApplicationRunListener的contextPrepared()方法。

9. 最核心的一步，将之前通过@EnableAutoConfiguration获取的所有配置以及其他形式的IoC容器配置加载到已经准备完毕的ApplicationContext。

10. 遍历调用所有SpringApplicationRunListener的contextLoaded()方法。

11. 调用ApplicationContext的refresh()方法，完成IoC容器可用的最后一道工序。

12. 查找当前ApplicationContext中是否注册有CommandLineRunner，如果有，则遍历执行它们。

13. 正常情况下，遍历执行SpringApplicationRunListener的finished()方法、（如果整个过程出现异常，则依然调用所有SpringApplicationRunListener的finished()方法，只不过这种情况下会将异常信息一并传入处理）

# 执行流程图解
整个过程看起来冗长无比,但其实很多都是一些事件通知的扩展点,如果我们将这些逻辑暂时忽略,执行流程如下图所示

![SpringBoot执行流程](/images/SpringBoot执行流程.jpg)

# 源码对应上述流程
## new SpringApplication(Application.class)源码
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

        this.webEnvironment = this.deduceWebEnvironment(); //判断是否为Web环境 根据"javax.servlet.Servlet", "org.springframework.web.context.ConfigurableWebApplicationContext"判断 对应1-1
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class)); //对应1-2
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class)); 
        this.mainApplicationClass = this.deduceMainApplicationClass(); //上面两行对应1-3 加载listener并推断main的定义类
    }
```
## springApplication.run()
```java
public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        Object analyzers = null;
        this.configureHeadlessProperty();
        SpringApplicationRunListeners listeners = this.getRunListeners(args); //
        listeners.started();

        try {
            DefaultApplicationArguments ex = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, ex);
            Banner printedBanner = this.printBanner(environment);
            context = this.createApplicationContext();
            new FailureAnalyzers(context);
            this.prepareContext(context, environment, listeners, ex, printedBanner);
            this.refreshContext(context);
            this.afterRefresh(context, ex);
            listeners.finished(context, (Throwable)null);
            stopWatch.stop();
            if(this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }

            return context;
        } catch (Throwable var9) {
            this.handleRunFailure(context, listeners, (FailureAnalyzers)analyzers, var9);
            throw new IllegalStateException(var9);
        }
    }
```

~~~源码对应上述过程~~

# 参考
[SpringBoot揭秘快速构建为服务体系][SpringBoot揭秘快速构建为服务体系]

[SpringBoot揭秘快速构建为服务体系]: https://yuedu.baidu.com/ebook/f945564883d049649a665824?pn=1&click_type=10010002&rf=https%3A%2F%2Fwww.google.com.hk%2F