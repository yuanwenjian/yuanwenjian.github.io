---
title: Mybatis源码阅读
description:  Mybatis源码阅读
date: 2018-11-14 20:40:00
comments: true
tags: 
    - Java  
categories:
    - Mybatis
---

# Mybatis-config配置源码


## 代码应用
```java
    private static SqlSessionFactoryBuilder sqlSessionFactoryBuilder;

    private static SqlSessionFactory sqlSessionFactory;

    public static void init() {
        String resource = "mybatis-config.xml";
        try {
            Reader reader = Resources.getResourceAsReader(resource);
            sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder(); //创建sqlSessionFactoryBuilder
            sqlSessionFactory = sqlSessionFactoryBuilder.build(reader);  //配置
            SqlSession sqlSession = sqlSessionFactory.openSession();
            PhoneImeiMapper mapper = sqlSession.getMapper(PhoneImeiMapper.class);
            List<PhoneImei> phoneImeiList = mapper.selectAll();
            System.out.println(phoneImeiList.size());
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    public static void main(String[] args) {
        init();
    }
```

##  xml配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <properties resource="config/generator.properties"/>

    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

    

    <typeAliases>
        <package name="com.yuanwj.mybatisdemo.model"/>
    </typeAliases>


    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
                <property name="driver" value="${jdbc.driver}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mybatis/mapper/PhoneImeiMapper.xml"/>
    </mappers>


</configuration>

```


以上为mybatis-config大概配置,通过官网可知configuration节点可配置以下节点  (properties?, settings?, typeAliases?, typeHandlers?, objectFactory?, objectWrapperFactory?, reflectorFactory?, plugins?, environments?, databaseIdProvider?, mappers?> ，下面分析源码配置

###  properties 节点
properties属性，可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递

如果属性在不只一个地方进行了配置，那么 MyBatis 将按照下面的顺序来加载：

在 properties 元素体内指定的属性首先被读取。
然后根据 properties 元素中的 resource 属性读取类路径下属性文件或根据 url 属性指定的路径读取属性文件，并覆盖已读取的同名属性。
最后读取作为方法参数传递的属性，并覆盖已读取的同名属性。
因此，通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的是 properties 属性中指定的属性。

属性优先级先记住，后面再分析源码实现


## 源码分析

### 1.1 configuration节点配置

```java
    // 创建SqlSessionFactory关键方法  位于SqlSessionFactoryBuilder类
    public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
        SqlSessionFactory var5;
        try {
            XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
            var5 = this.build(parser.parse());//关键方法
        } catch (Exception var14) {
            throw ExceptionFactory.wrapException("Error building SqlSession.", var14);
        } finally {
            ErrorContext.instance().reset();

            try {
                reader.close();
            } catch (IOException var13) {
                ;
            }

        }
        return var5;
    }

    // parse核心方法 位于XMLConfigBuilder

    public Configuration parse() {
        if (this.parsed) {
            throw new BuilderException("Each XMLConfigBuilder can only be used once.");
        } else {
            this.parsed = true;
            this.parseConfiguration(this.parser.evalNode("/configuration"));
            return this.configuration;
        }
    }
    //这一步与之前configuration规定子节点对应
    private void parseConfiguration(XNode root) {
        try {
            this.propertiesElement(root.evalNode("properties")); //解析properties文件
            Properties settings = this.settingsAsProperties(root.evalNode("settings")); //解析setting节点
            this.loadCustomVfs(settings);
            this.typeAliasesElement(root.evalNode("typeAliases"));
            this.pluginElement(root.evalNode("plugins"));
            this.objectFactoryElement(root.evalNode("objectFactory"));
            this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
            this.reflectorFactoryElement(root.evalNode("reflectorFactory"));
            this.settingsElement(settings);
            this.environmentsElement(root.evalNode("environments"));
            this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
            this.typeHandlerElement(root.evalNode("typeHandlers"));
            this.mapperElement(root.evalNode("mappers"));
        } catch (Exception var3) {
            throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
        }
    }
```

#### 1.2 首先分析properties节点
```java
    //XMLConfigBuilder 类
    private void propertiesElement(XNode context) throws Exception {
        if (context != null) {
            Properties defaults = context.getChildrenAsProperties(); //该方法为读取配置 及读取properties节点下的属性
            String resource = context.getStringAttribute("resource"); //改行与下一行为读取resource，url属性
            String url = context.getStringAttribute("url");
            if (resource != null && url != null) {
                throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
            }
            //如果不为空，读取配置，假如与之前properties有相同的配置，则覆盖，与上面优先级对应
            if (resource != null) {
                defaults.putAll(Resources.getResourceAsProperties(resource));
            } else if (url != null) {
                defaults.putAll(Resources.getUrlAsProperties(url));
            }

            Properties vars = this.configuration.getVariables();
            if (vars != null) {
                defaults.putAll(vars);
            }

            this.parser.setVariables(defaults);
            this.configuration.setVariables(defaults);
        }
    }

    //XNode 类，解析xml属性 经常用到
    public Properties getChildrenAsProperties() {
        Properties properties = new Properties();
        Iterator var2 = this.getChildren().iterator();

        while(var2.hasNext()) {
            XNode child = (XNode)var2.next();
            String name = child.getStringAttribute("name");
            String value = child.getStringAttribute("value");
            if (name != null && value != null) {
                properties.setProperty(name, value);
            }
        }

        return properties;
    }
```

#### 1.2 setting节点
这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 允许配置可查看[mybatis官网][mybatis官网] ,官网内容更加详细，本段主要分析configuration是如何控制参数的设置,简单来说就是通过反射判断Configuration类是否有参数的setter方法，以下为具体实现
```java
    
    private Properties settingsAsProperties(XNode context) {
        if (context == null) {
            return new Properties();
        } else {
            Properties props = context.getChildrenAsProperties();
            MetaClass metaConfig = MetaClass.forClass(Configuration.class, this.localReflectorFactory);
            Iterator var4 = props.keySet().iterator();

            Object key;
            do {
                if (!var4.hasNext()) {
                    return props;
                }

                key = var4.next();
            } while(metaConfig.hasSetter(String.valueOf(key)));  //判断是否有配置参数的set方法

            throw new BuilderException("The setting " + key + " is not known.  Make sure you spelled it correctly (case sensitive).");
        }
    }
    // 通过idea，查询调用方法链，以下仅标注主要方法，
    // MetaClass类，注意此时泛型type实际为上文中的Configuration.class
    private MetaClass(Class<?> type, ReflectorFactory reflectorFactory) {
        this.reflectorFactory = reflectorFactory;
        this.reflector = reflectorFactory.findForClass(type);
    }

    public static MetaClass forClass(Class<?> type, ReflectorFactory reflectorFactory) {
        return new MetaClass(type, reflectorFactory);
    }

    // DefaultReflectorFactory 类
    public Reflector findForClass(Class<?> type) {
        if (this.classCacheEnabled) {
            Reflector cached = (Reflector)this.reflectorMap.get(type);
            if (cached == null) {
                cached = new Reflector(type);
                this.reflectorMap.put(type, cached);
            }

            return cached;
        } else {
            return new Reflector(type);
        }
    }

    // Reflector类 此时泛型为Configuration.class ，
    public Reflector(Class<?> clazz) {
        this.type = clazz;
        this.addDefaultConstructor(clazz);
        this.addGetMethods(clazz);
        this.addSetMethods(clazz); // 添加类的set方法
        this.addFields(clazz);
        this.readablePropertyNames = (String[])this.getMethods.keySet().toArray(new String[this.getMethods.keySet().size()]);
        this.writeablePropertyNames = (String[])this.setMethods.keySet().toArray(new String[this.setMethods.keySet().size()]);
        String[] var2 = this.readablePropertyNames;
        int var3 = var2.length;

        int var4;
        String propName;
        for(var4 = 0; var4 < var3; ++var4) {
            propName = var2[var4];
            this.caseInsensitivePropertyMap.put(propName.toUpperCase(Locale.ENGLISH), propName);
        }

        var2 = this.writeablePropertyNames;
        var3 = var2.length;

        for(var4 = 0; var4 < var3; ++var4) {
            propName = var2[var4];
            this.caseInsensitivePropertyMap.put(propName.toUpperCase(Locale.ENGLISH), propName);
        }
    }

    // 以上就是添加Configuration.class的所有set方法，下面就是看是否存在参数配置的set

    // MetaClass
    public boolean hasSetter(String name) {
        PropertyTokenizer prop = new PropertyTokenizer(name);
        if (prop.hasNext()) {
            if (this.reflector.hasSetter(prop.getName())) {
                MetaClass metaProp = this.metaClassForProperty(prop.getName());
                return metaProp.hasSetter(prop.getChildren());
            } else {
                return false;
            }
        } else {
            return this.reflector.hasSetter(prop.getName());
        }
    }
    // Reflector 
    public boolean hasSetter(String propertyName) {
        return this.setMethods.keySet().contains(propertyName);
    }
```

# 参考

1. [mybatis官网][mybatis官网]

2. [项目代码][代码]

[mybatis官网]: http://www.mybatis.org/mybatis-3/zh/configuration.html

[代码]:https://github.com/yuanwenjian/mybatis-code