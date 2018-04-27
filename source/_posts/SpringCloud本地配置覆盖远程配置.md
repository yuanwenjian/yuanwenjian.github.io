---
title: SpringCloud Config 配置文件与本地配置优先级
description: 
date: 2018-04-27 20:40:00
comments: true
tags: 
    - Spring
    - Java
categories:
    - Spring
---
# SpringCloudConfig远程配置与项目本地application配置优先级

## 问题原因
今天启动项目，发现端口占用,改完本地端口配置后无效，解决办法为将之前端口停掉，但是意识到本地无法覆盖远程配置，所以研究一下

## 源码

```java
//Spring-Cloud-context-1.2.4.RELEASE  PropertySourceBootstrapConfiguration.class 核心逻辑判断
private void insertPropertySources(MutablePropertySources propertySources, CompositePropertySource composite) {
    MutablePropertySources incoming = new MutablePropertySources();
    incoming.addFirst(composite);
    PropertySourceBootstrapProperties remoteProperties = new PropertySourceBootstrapProperties();
    (new RelaxedDataBinder(remoteProperties, "spring.cloud.config")).bind(new PropertySourcesPropertyValues(incoming));
    if (remoteProperties.isAllowOverride() && (remoteProperties.isOverrideNone() || !remoteProperties.isOverrideSystemProperties())) {
        if (remoteProperties.isOverrideNone()) { //我的判断为overrideNone 设为true 加入最后，当取值时，遍历propertySourceList，如果之前属性存在，则使用，所以本地会覆盖远程
            propertySources.addLast(composite);
        } else {
            if (propertySources.contains("systemEnvironment")) {
                if (!remoteProperties.isOverrideSystemProperties()) { 
                    propertySources.addAfter("systemEnvironment", composite);
                } else {//默认 
                    propertySources.addBefore("systemEnvironment", composite); 
                }
            } else {
                propertySources.addLast(composite);
            }

        }
    } else { //默认
        propertySources.addFirst(composite);
    }
}

//Spring-Cloud-context-1.2.4.RELEASE  MutablePropertySources.class 取值
public PropertySource<?> get(String name) {
    int index = this.propertySourceList.indexOf(PropertySource.named(name));
    return index != -1 ? (PropertySource)this.propertySourceList.get(index) : null;
}

//Spring-Cloud-context-1.2.4.RELEASE  PropertySourceBootstrapProperties.class  默认属性
public class PropertySourceBootstrapProperties {
    private boolean overrideSystemProperties = true; //允许覆盖系统属性，及java -Dxxx传入
    private boolean allowOverride = true; //允许本地覆盖 
    private boolean overrideNone = false; //允许本地覆盖总判断
}
```

## 远程覆盖配置
```yml
spring:
  cloud:
    config:
      allowOverride: true
      overrideNone: true
      overrideSystemProperties: false
```

## 注意

以上配置放入远程配置文件中才有效，即SpringCloudConfig的配置文件，在本地添加无效




# 参考
[Spring Cloud Config覆盖][SpringCloud]

[SpringCloud]:http://www.cnblogs.com/xingxueliao/p/7113651.html