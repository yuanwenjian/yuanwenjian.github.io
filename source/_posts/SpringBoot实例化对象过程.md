---
title: SpringBoot对象实例化步骤
description: SpringBoot对象实例化步骤
date: 2018-05-03 13:59:00
comments: true
tags: 
    - Spring  
    - SpringBoot
categories:
    - Spring
    - SpringBoot
---

SpringBoot 1.5.7

1. 创建ConfigurableApplicationContext 对象
```java
SpringApplication 315行
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
创建context类型为AnnotationConfigEmbeddedWebApplicationContext,内置beanfactory为DefaultListableBeanFactory
类图如下
![webapplication][webapplication]
```java
类图中GenericApplicationContext无参构造方法如下所示
public GenericApplicationContext() {
    this.customClassLoader = false;
    this.refreshed = new AtomicBoolean();
    this.beanFactory = new DefaultListableBeanFactory();
}
```
```java
步骤    行数    方法名
SpringApplication
1. 147   SpringApplication.run()   
2. 163   SpringApplication.refreshContext(context)
3. 211   this.refresh(context);
4. 427   ((AbstractApplicationContext)applicationContext).refresh()

AbstractApplicationContext
5. 269   this.finishBeanFactoryInitialization(beanFactory);
6. 477   this.getBean(weaverAwareName);
7. 586   this.getBeanFactory().getBean(name);

AbstractBeanFactory
8. 97 this.doGetBean(name, (Class)null, (Object[])null, false)
9. 1123 protected abstract Object createBean(String var1, RootBeanDefinition var2, Object[] var3)

AbstractAutowireCapableBeanFactory
10. 306 this.doCreateBean(beanName, mbdToUse, args)
11. 321 this.createBeanInstance(beanName, mbd, args)
12. 731  autowireNecessary ? this.autowireConstructor(beanName, mbd, (Constructor[])null, (Object[])null) : this.instantiateBean(beanName, mbd);
13. 783 (new ConstructorResolver(this)).autowireConstructor(beanName, mbd, ctors, explicitArgs)

ConstructorResolver
14. 203 this.beanFactory.getInstantiationStrategy().instantiate(mbd, beanName, this.beanFactory, constructorToUse, argsToUse);

SimpleInstantiationStrategy
15. 82  BeanUtils.instantiateClass(ctor, args)

BeanUtils
16. 76 ctor.newInstance(args)

```


[webapplication]: /images/webapplication.png