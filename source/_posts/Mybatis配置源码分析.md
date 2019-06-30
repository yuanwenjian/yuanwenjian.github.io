---
title: Mybatis源码阅读
description:  Mybatis源码阅读
date: 2019-04-23 20:40:00
comments: true
tags: 
    - Java  
    - Mybatis 
categories:
    - Mybatis
    - Java 
---

# Mybatis-config配置源码

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。源码基于mybatis3.4.6版本,并且以下源码仅为mybatis单独使用，不包括与spring整合，spring整合mybatis还得考虑mapper配置装载到容器等，后续再进行分析

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


以上为mybatis-config大概配置,通过官网可知configuration节点可配置以下节点  (properties?, settings?, typeAliases?, typeHandlers?, objectFactory?, objectWrapperFactory?, reflectorFactory?, plugins?, environments?, databaseIdProvider?, mappers?> ，并且节点位置是有顺序的，否则会报错，顺序如上所述 下面分析源码配置



## 源码分析

### 1.1 configuration节点配置

```java
    // 创建SqlSessionFactory关键方法  位于SqlSessionFactoryBuilder类
    public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
        SqlSessionFactory var5;
        try {
            XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);//创建XMLConfigBuilder
            var5 = this.build(parser.parse());//关键方法用来解析配置
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
        if (this.parsed) {//如果已经解析过了抛异常
            throw new BuilderException("Each XMLConfigBuilder can only be used once.");
        } else {
            this.parsed = true;
            this.parseConfiguration(this.parser.evalNode("/configuration"));// 解析configuration节点
            return this.configuration;
        }
    }
    //这一步与之前configuration规定子节点对应，该方法逐步解析各个节点
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

### 1.2 首先分析properties节点
properties属性，可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递

如果属性在不只一个地方进行了配置，那么 MyBatis 将按照下面的顺序来加载：

在 properties 元素体内指定的属性首先被读取。
然后根据 properties 元素中的 resource 属性读取类路径下属性文件或根据 url 属性指定的路径读取属性文件，并覆盖已读取的同名属性。
最后读取作为方法参数传递的属性，并覆盖已读取的同名属性。
因此，通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的是 properties 属性中指定的属性。

```java
    //XMLConfigBuilder 类
    private void propertiesElement(XNode context) throws Exception {
        if (context != null) {
            Properties defaults = context.getChildrenAsProperties(); //该方法为读取配置 及读取properties节点下的属性
            String resource = context.getStringAttribute("resource"); //改行与下一行为读取resource，url属性
            String url = context.getStringAttribute("url");
            if (resource != null && url != null) {//如果两个都未配置抛异常
                throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
            }
            //如果不为空，读取配置，假如与之前properties有相同的配置，则覆盖，与上面优先级对应
            if (resource != null) {
                defaults.putAll(Resources.getResourceAsProperties(resource));//参数为resource时解析
            } else if (url != null) {
                defaults.putAll(Resources.getUrlAsProperties(url)); //url 解析
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

### 1.3 setting节点
这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 允许配置可查看[mybatis官网][mybatis官网] ,官网内容更加详细，本段主要分析configuration是如何控制参数的设置,简单来说就是通过反射判断Configuration类是否有参数的setter方法，以下为具体实现
```java
    
    private Properties settingsAsProperties(XNode context) {
        if (context == null) {
            return new Properties(); //未配置返回空
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
            } while(metaConfig.hasSetter(String.valueOf(key)));  //判断是否有配置参数的set方法，这段代码判断Configuration.class是否存在
            //如果设置中参数中不存在setter方法，抛异常
            throw new BuilderException("The setting " + key + " is not known.  Make sure you spelled it correctly (case sensitive).");
        }
    }
    // 通过idea，查询方法链，以下仅标注主要方法，
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
        if (this.classCacheEnabled) {// 猜测为缓存，具体没有研究
            Reflector cached = (Reflector)this.reflectorMap.get(type);
            if (cached == null) {
                cached = new Reflector(type);//调用构造方法
                this.reflectorMap.put(type, cached); //放入缓存？
            }

            return cached;
        } else {
            return new Reflector(type);//调用构造方法
        }
    }

    // Reflector类 构造方法 此时泛型为Configuration.class ，
    public Reflector(Class<?> clazz) {
        this.type = clazz;
        this.addDefaultConstructor(clazz);
        this.addGetMethods(clazz);
        this.addSetMethods(clazz); // 添加该类所有set方法
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

    // 以上就是添加Configuration.class的所有set方法，下面就是看是否存在参数配置的set方法，如果没有，启动失败，

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

### 1.4 typeAliases节点
类型别名是为 Java 类型设置一个短的名字。 它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余，Mybatis自带常见的 Java 类型内建的相应的类型别名。它们都是不区分大小写的，注意对基本类型名称重复采取的特殊命名风格。具体可通过官网查看[mybatis内置别名][mybatis内置别名]

xml配置如下
```xml
    <typeAliases>
      <typeAlias alias="Author" type="domain.blog.Author"/>
    </typeAliases>
    <!-- 或者使用包名 MyBatis 会在包名下面搜索需要的 Java Bean， 每一个在包 com.yuanwj.mybatisdemo.model 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 domain.blog.Author 的别名为 author；若有注解，则别名为其注解值-->

    <typeAliases>
        <package name="com.yuanwj.mybatisdemo.model"/>
    </typeAliases>
```
源码实现
```java
    // XMLConfigBuilder
    private void typeAliasesElement(XNode parent) {
        if (parent != null) {
            Iterator i$ = parent.getChildren().iterator();//获取子节点

            while(i$.hasNext()) {
                XNode child = (XNode)i$.next();
                String alias;
                if ("package".equals(child.getName())) { //如果配置package,则扫描包下所有类
                    alias = child.getStringAttribute("name");
                    this.configuration.getTypeAliasRegistry().registerAliases(alias); //注册alias
                } else {
                    alias = child.getStringAttribute("alias"); //读取alias属性
                    String type = child.getStringAttribute("type");

                    try {
                        Class<?> clazz = Resources.classForName(type);//获取类名
                        if (alias == null) {
                            this.typeAliasRegistry.registerAlias(clazz);//未设置别名，使用类名代替
                        } else {
                            this.typeAliasRegistry.registerAlias(alias, clazz);//注册alias
                        }
                    } catch (ClassNotFoundException var7) { //未找到抛异常
                        throw new BuilderException("Error registering typeAlias for '" + alias + "'. Cause: " + var7, var7);
                    }
                }
            }
        }
    }

    // TypeAliasRegistry
    public void registerAliases(String packageName) {
        this.registerAliases(packageName, Object.class);
    }

    public void registerAliases(String packageName, Class<?> superType) {
        ResolverUtil<Class<?>> resolverUtil = new ResolverUtil();
        resolverUtil.find(new IsA(superType), packageName);//获取package下所有类
        Set<Class<? extends Class<?>>> typeSet = resolverUtil.getClasses(); //反射获取包下所有class
        Iterator i$ = typeSet.iterator();

        while(i$.hasNext()) {
            Class<?> type = (Class)i$.next();
            if (!type.isAnonymousClass() && !type.isInterface() && !type.isMemberClass()) { //如果不是匿名类，不是接口，不是成员类，添加
                this.registerAlias(type); //注册 查看registerAlias(Class<?> type)方法调用
            }
        }
    }    

    public void registerAlias(Class<?> type) {
        String alias = type.getSimpleName();//获取类名
        Alias aliasAnnotation = (Alias)type.getAnnotation(Alias.class);//获取Alias注解
        if (aliasAnnotation != null) {//如果有注解，别名会被注解名称替换
            alias = aliasAnnotation.value();
        }

        this.registerAlias(alias, type);
    }

    public void registerAlias(String alias, Class<?> value) {
        if (alias == null) {
            throw new TypeException("The parameter alias cannot be null");
        } else {
            String key = alias.toLowerCase(Locale.ENGLISH);//改为小写
            if (this.TYPE_ALIASES.containsKey(key) && this.TYPE_ALIASES.get(key) != null && !((Class)this.TYPE_ALIASES.get(key)).equals(value)) {//如果已经别名已经被使用，抛异常
                throw new TypeException("The alias '" + alias + "' is already mapped to the value '" + ((Class)this.TYPE_ALIASES.get(key)).getName() + "'.");
            } else {
                this.TYPE_ALIASES.put(key, value);//放入TYPE_ALIASES,TYPE_ALIASES类型为Map,以下代码可查看内置别名，与官网对应
            }
        }
    }
    //内置别名
    public TypeAliasRegistry() {
        this.registerAlias("string", String.class);
        this.registerAlias("byte", Byte.class);
        this.registerAlias("long", Long.class);
        this.registerAlias("short", Short.class);
        this.registerAlias("int", Integer.class);
        this.registerAlias("integer", Integer.class);
        this.registerAlias("double", Double.class);
        this.registerAlias("float", Float.class);
        this.registerAlias("boolean", Boolean.class);
        this.registerAlias("byte[]", Byte[].class);
        this.registerAlias("long[]", Long[].class);
        this.registerAlias("short[]", Short[].class);
        this.registerAlias("int[]", Integer[].class);
        this.registerAlias("integer[]", Integer[].class);
        this.registerAlias("double[]", Double[].class);
        this.registerAlias("float[]", Float[].class);
        this.registerAlias("boolean[]", Boolean[].class);
        this.registerAlias("_byte", Byte.TYPE);
        this.registerAlias("_long", Long.TYPE);
        this.registerAlias("_short", Short.TYPE);
        this.registerAlias("_int", Integer.TYPE);
        this.registerAlias("_integer", Integer.TYPE);
        this.registerAlias("_double", Double.TYPE);
        this.registerAlias("_float", Float.TYPE);
        this.registerAlias("_boolean", Boolean.TYPE);
        this.registerAlias("_byte[]", byte[].class);
        this.registerAlias("_long[]", long[].class);
        this.registerAlias("_short[]", short[].class);
        this.registerAlias("_int[]", int[].class);
        this.registerAlias("_integer[]", int[].class);
        this.registerAlias("_double[]", double[].class);
        this.registerAlias("_float[]", float[].class);
        this.registerAlias("_boolean[]", boolean[].class);
        this.registerAlias("date", Date.class);
        this.registerAlias("decimal", BigDecimal.class);
        this.registerAlias("bigdecimal", BigDecimal.class);
        this.registerAlias("biginteger", BigInteger.class);
        this.registerAlias("object", Object.class);
        this.registerAlias("date[]", Date[].class);
        this.registerAlias("decimal[]", BigDecimal[].class);
        this.registerAlias("bigdecimal[]", BigDecimal[].class);
        this.registerAlias("biginteger[]", BigInteger[].class);
        this.registerAlias("object[]", Object[].class);
        this.registerAlias("map", Map.class);
        this.registerAlias("hashmap", HashMap.class);
        this.registerAlias("list", List.class);
        this.registerAlias("arraylist", ArrayList.class);
        this.registerAlias("collection", Collection.class);
        this.registerAlias("iterator", Iterator.class);
        this.registerAlias("ResultSet", ResultSet.class);
    }    
```

### 1.5 plugins节点

MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
ParameterHandler (getParameterObject, setParameters)
ResultSetHandler (handleResultSets, handleOutputParameters)
StatementHandler (prepare, parameterize, batch, update, query)

只分析配置，插件实现后续分析
xml配置
```xml
<plugins>
    <!-- 插件路径 -->
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <!-- 参数配置 -->
        <property name="param1" value="value1"/>
	</plugin>
</plugins>
```

源码实现
```java
    private void pluginElement(XNode parent) throws Exception {
        if (parent != null) {
            Iterator i$ = parent.getChildren().iterator();

            while(i$.hasNext()) {
                XNode child = (XNode)i$.next();
                String interceptor = child.getStringAttribute("interceptor"); //获取插件路径
                Properties properties = child.getChildrenAsProperties();
                Interceptor interceptorInstance = (Interceptor)this.resolveClass(interceptor).newInstance(); //反射生成实例
                interceptorInstance.setProperties(properties); //配置属性
                this.configuration.addInterceptor(interceptorInstance); //添加到configuration
            }
        }

    }
```
### 1.6 environments节点
MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 Schema 的多个生产数据库中 使用相同的 SQL 映射。有许多类似的使用场景。

不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。

所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推，记起来很简单：

每个数据库对应一个 SqlSessionFactory 实例


xml配置

```xml
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
```
java实现代码

```java
    //XMLConfigBuilder.class 解析environment节点
    private void environmentsElement(XNode context) throws Exception {
        if (context != null) {
            if (this.environment == null) {
                this.environment = context.getStringAttribute("default"); //为配置使用默认
            }

            Iterator i$ = context.getChildren().iterator();

            while(i$.hasNext()) {
                XNode child = (XNode)i$.next();
                String id = child.getStringAttribute("id"); //获取id配置
                if (this.isSpecifiedEnvironment(id)) {
                    TransactionFactory txFactory = this.transactionManagerElement(child.evalNode("transactionManager")); //获取事务管理
                    DataSourceFactory dsFactory = this.dataSourceElement(child.evalNode("dataSource")); //配置数据源
                    DataSource dataSource = dsFactory.getDataSource(); //获取数据源
                    Builder environmentBuilder = (new Builder(id)).transactionFactory(txFactory).dataSource(dataSource);
                    this.configuration.setEnvironment(environmentBuilder.build());
                }
            }
        }
    }
    //XMLConfigBuilder.class  生成DatasourceFactory
    private DataSourceFactory dataSourceElement(XNode context) throws Exception {
        if (context != null) {
            //解析type,有三种内建的数据源类型（也就是 type=”[UNPOOLED|POOLED|JNDI]”）：
            //UNPOOLED– 这个数据源的实现只是每次被请求时打开和关闭连接。虽然有点慢，但对于在数据库连接可用性方面没有太高要求的简单应用程序来说，是一个很好的选择。 不同的数据库在性能方面的表现也是不一样的，对于某些数据库来说，使用连接池并不重要
            //POOLED– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这是一种使得并发 Web 应用快速响应请求的流行处理方式。
            //JNDI – 这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。
            String type = context.getStringAttribute("type"); 
            Properties props = context.getChildrenAsProperties();
            DataSourceFactory factory = (DataSourceFactory)this.resolveClass(type).newInstance(); // 通过反射生成mybatis 内置UNPOOLED，POOLED,JNDI，分别对应不同DatasourceFactory，本文以PooledDataSourceFactory为例，查看代码
            factory.setProperties(props); //设置属性
            return factory;
        } else {
            throw new BuilderException("Environment declaration requires a DataSourceFactory.");
        }
    }    
    //通过别名生成class，
    protected Class<?> resolveClass(String alias) {
        if (alias == null) {
            return null;
        } else {
            try {
                return this.resolveAlias(alias);
            } catch (Exception var3) {
                throw new BuilderException("Error resolving class. Cause: " + var3, var3);
            }
        }
    }
//PooledDataSourceFactory.class 继承于UnpooledDataSourceFactory
public class PooledDataSourceFactory extends UnpooledDataSourceFactory {
    //构造方法
    public PooledDataSourceFactory() {
        this.dataSource = new PooledDataSource();
    }
}
    //PooledDataSource构造方法，创建的是UnpooledDataSource 类型数据源
    public PooledDataSource() {
        this.dataSource = new UnpooledDataSource();
    }    

    // UnpooledDataSourceFactory.class 设置属性方法，同解析setting类似，通过反射，判断是否包含setter方法并设置属性
    public void setProperties(Properties properties) {
        Properties driverProperties = new Properties();
        MetaObject metaDataSource = SystemMetaObject.forObject(this.dataSource); // 连同上面代码，datasource类型为UnpooledDataSource 类型
        Iterator i$ = properties.keySet().iterator();

        while(i$.hasNext()) {
            Object key = i$.next();
            String propertyName = (String)key;
            String value;
            //推测为如果以driver.开头，设置驱动的属性
            if (propertyName.startsWith("driver.")) {
                value = properties.getProperty(propertyName);
                driverProperties.setProperty(propertyName.substring(DRIVER_PROPERTY_PREFIX_LENGTH), value);
            } else {
                if (!metaDataSource.hasSetter(propertyName)) { //判断是否含有配置的setter方法
                    throw new DataSourceException("Unknown DataSource property: " + propertyName);
                }

                value = (String)properties.get(propertyName);
                Object convertedValue = this.convertValue(metaDataSource, propertyName, value);//转换属性值
                metaDataSource.setValue(propertyName, convertedValue); //设置属性
            }
        }

        if (driverProperties.size() > 0) {
            metaDataSource.setValue("driverProperties", driverProperties); //设置driver属性
        }

    }
```



### 1.7 mappers节点

我们现在就要定义 SQL 映射语句了。 但是首先我们需要告诉 MyBatis 到哪里去找到这些语句。 Java 在自动查找这方面没有提供一个很好的方法，所以最佳的方式是告诉 MyBatis 到哪里去找映射文件。本文只解析配置mapper加载过程，至于如何通过mapper访问数据库，后续再分析，配置mapper，结果简单形容为，将mapper类设置到configuration中this.knownMappers参数中，后续执行时再从knownMappers中获取对应的mapper

xml配置

```xml
    <!-- 使用相对于类路径的资源引用 -->
    <mappers>
        <mapper resource="mybatis/mapper/PhoneImeiMapper.xml"/>
    </mappers>

    <!-- 使用完全限定资源定位符（URL） -->
    <mappers>
        <mapper url="file:///mybatis/mapper/PhoneImeiMapper.xml"/>
    </mappers>
    <!-- 使用映射器接口实现类的完全限定类名 -->
    <mappers>
        <mapper class="mybatis/mapper/PhoneImeiMapper.xml"/>
    </mappers>

    <!-- 将包内的映射器接口实现全部注册为映射器 -->
    <mappers>
         <package name="mybatis.mapper"/>
    </mappers>

```

java实现
```java
    //XMLConfigBuilder.class 解析mapper节点
    private void mapperElement(XNode parent) throws Exception {
        if (parent != null) {
            Iterator var2 = parent.getChildren().iterator();

            while(true) {
                while(var2.hasNext()) {
                    XNode child = (XNode)var2.next(); //遍历所有子节点
                    String resource;
                    if ("package".equals(child.getName())) { // 如果配置package,扫描包下所有class添加到configuration mappers
                        resource = child.getStringAttribute("name");
                        this.configuration.addMappers(resource); //将配置包的mapper添加到configuration中、具体实现下面配置添加到configuration中
                    } else {
                        resource = child.getStringAttribute("resource");//不同参数，使用不同方法,主要分析参数为resource与url源码实现
                        String url = child.getStringAttribute("url");
                        String mapperClass = child.getStringAttribute("class");
                        XMLMapperBuilder mapperParser;
                        InputStream inputStream;
                        if (resource != null && url == null && mapperClass == null) {//参数为resource
                            ErrorContext.instance().resource(resource);
                            inputStream = Resources.getResourceAsStream(resource);
                            mapperParser = new XMLMapperBuilder(inputStream, this.configuration, resource, this.configuration.getSqlFragments());//创建XMLMapperBuilder,设置对应属性，用来解析mapper
                            mapperParser.parse();//关键方法，解析mapper
                        } else if (resource == null && url != null && mapperClass == null) {//参数为url与解析resource类型
                            ErrorContext.instance().resource(url);
                            inputStream = Resources.getUrlAsStream(url);
                            mapperParser = new XMLMapperBuilder(inputStream, this.configuration, url, this.configuration.getSqlFragments());//创建XMLMapperBuilder,用来解析mapper
                            mapperParser.parse();//解析mapper
                        } else {
                            if (resource != null || url != null || mapperClass == null) { //参数为class
                                throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
                            }

                            Class<?> mapperInterface = Resources.classForName(mapperClass);
                            this.configuration.addMapper(mapperInterface); //与解析package类似，直接放入configuration
                        }
                    }
                }

                return;
            }
        }
    }
```

配置添加到configuration中 源码

```java
    //Configuration.class
    public void addMappers(String packageName) {
        this.mapperRegistry.addMappers(packageName);
    }
    //MapperRegistry.class
    public void addMappers(String packageName) {
        this.mapperRegistry.addMappers(packageName);
    }
    public void addMappers(String packageName, Class<?> superType) {
        ResolverUtil<Class<?>> resolverUtil = new ResolverUtil();
        resolverUtil.find(new IsA(superType), packageName);
        Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();//获取package下所有mapper class
        Iterator var5 = mapperSet.iterator();

        while(var5.hasNext()) { //遍历
            Class<?> mapperClass = (Class)var5.next();
            this.addMapper(mapperClass); // 添加到配置中
        }
    }


    public <T> void addMapper(Class<T> type) {
        if (type.isInterface()) {
            if (this.hasMapper(type)) { //如果当前已有类型，抛异常
                throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
            }

            boolean loadCompleted = false;

            try {
                this.knownMappers.put(type, new MapperProxyFactory(type)); //生成MapperProxyFactory放入knownMappers中，执行时根据类名取出
                MapperAnnotationBuilder parser = new MapperAnnotationBuilder(this.config, type);
                parser.parse();
                loadCompleted = true;
            } finally {
                if (!loadCompleted) {
                    this.knownMappers.remove(type);
                }

            }
        }
//MapperProxyFactory 类是 生产 Mapper 代理类的工厂，用 Java 动态代理实现：
public class MapperProxyFactory<T> {
    private final Class<T> mapperInterface;
    private final Map<Method, MapperMethod> methodCache = new ConcurrentHashMap();

    public MapperProxyFactory(Class<T> mapperInterface) {//构造方法，参数mapperInterface 值为对应的mapper.class,获取mapper实例时需要
        this.mapperInterface = mapperInterface;
    }

    public Class<T> getMapperInterface() {
        return this.mapperInterface;
    }

    public Map<Method, MapperMethod> getMethodCache() {
        return this.methodCache;
    }

    protected T newInstance(MapperProxy<T> mapperProxy) {
        return Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
    }

    public T newInstance(SqlSession sqlSession) {
        MapperProxy<T> mapperProxy = new MapperProxy(sqlSession, this.mapperInterface, this.methodCache);
        return this.newInstance(mapperProxy);
    }
}

```

mapper参数为resource或url解析源码，创建XMLMapperBuilder并解析mapper源码，
```java
    //XMLMapperBuilder.class XMLMapperBuilder构造方法，并设置几个属性
    private XMLMapperBuilder(XPathParser parser, Configuration configuration, String resource, Map<String, XNode> sqlFragments) {
        super(configuration);
        this.builderAssistant = new MapperBuilderAssistant(configuration, resource);
        this.parser = parser;
        this.sqlFragments = sqlFragments;
        this.resource = resource;
    }

    //XMLMapperBuilder.class 解析mapper
    public void parse() {
        if (!this.configuration.isResourceLoaded(this.resource)) { //如果之前未加载，则解析
            this.configurationElement(this.parser.evalNode("/mapper"));//解析mapper节点，
            this.configuration.addLoadedResource(this.resource);//添加到configuration中
            this.bindMapperForNamespace();//通过namespace绑定到configuration中kows_mapper中
        }

        //以下通过方法名推断为解析待处理resultmap,缓存等，具体实现未研究
        this.parsePendingResultMaps();
        this.parsePendingCacheRefs();
        this.parsePendingStatements();
    }

    //解析mapper节点
    private void configurationElement(XNode context) {
        try {
            String namespace = context.getStringAttribute("namespace");
            // 以下为解析mapper
            if (namespace != null && !namespace.equals("")) {
                this.builderAssistant.setCurrentNamespace(namespace);//设置namespace
                this.cacheRefElement(context.evalNode("cache-ref"));//解析cache-ref
                this.cacheElement(context.evalNode("cache"));
                this.parameterMapElement(context.evalNodes("/mapper/parameterMap"));
                this.resultMapElements(context.evalNodes("/mapper/resultMap"));
                this.sqlElement(context.evalNodes("/mapper/sql"));
                this.buildStatementFromContext(context.evalNodes("select|insert|update|delete")); //解析sql语句
            } else {
                throw new BuilderException("Mapper's namespace cannot be empty");
            }
        } catch (Exception var3) {
            throw new BuilderException("Error parsing Mapper XML. The XML location is '" + this.resource + "'. Cause: " + var3, var3);
        }
    }

    //将配置好的mapper设置到configuration中，通过namespace绑定
    private void bindMapperForNamespace() {
        String namespace = this.builderAssistant.getCurrentNamespace();//获取之前绑定namespace
        if (namespace != null) {
            Class boundType = null;

            try {
                boundType = Resources.classForName(namespace); //获取类名
            } catch (ClassNotFoundException var4) {
                ;
            }

            if (boundType != null && !this.configuration.hasMapper(boundType)) {//如果不为空，且当前配置不含这个mapper,添加，添加方法与配置添加到configuration中 源码一样
                this.configuration.addLoadedResource("namespace:" + namespace); //添加resource
                this.configuration.addMapper(boundType);  //将当前mapper添加到knownMappers中
            }
        }

    }
```

mapper解析sql语句

```java

    //XMLMapperBuilder.class 
    private void buildStatementFromContext(List<XNode> list) {
        if (this.configuration.getDatabaseId() != null) {
            this.buildStatementFromContext(list, this.configuration.getDatabaseId());
        }

        this.buildStatementFromContext(list, (String)null);
    }

    //XMLMapperBuilder.class 
    private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
        Iterator var3 = list.iterator();

        while(var3.hasNext()) {
            XNode context = (XNode)var3.next();
            XMLStatementBuilder statementParser = new XMLStatementBuilder(this.configuration, this.builderAssistant, context, requiredDatabaseId);

            try {
                statementParser.parseStatementNode();
            } catch (IncompleteElementException var7) {
                this.configuration.addIncompleteStatement(statementParser);
            }
        }

    }

    //XMLStatementBuilder
    public void parseStatementNode() {
        String id = this.context.getStringAttribute("id");
        String databaseId = this.context.getStringAttribute("databaseId");
        if (this.databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {
            Integer fetchSize = this.context.getIntAttribute("fetchSize");
            Integer timeout = this.context.getIntAttribute("timeout");
            String parameterMap = this.context.getStringAttribute("parameterMap");
            String parameterType = this.context.getStringAttribute("parameterType");
            Class<?> parameterTypeClass = this.resolveClass(parameterType);
            String resultMap = this.context.getStringAttribute("resultMap");
            String resultType = this.context.getStringAttribute("resultType");
            String lang = this.context.getStringAttribute("lang");
            LanguageDriver langDriver = this.getLanguageDriver(lang);
            Class<?> resultTypeClass = this.resolveClass(resultType);
            String resultSetType = this.context.getStringAttribute("resultSetType");
            StatementType statementType = StatementType.valueOf(this.context.getStringAttribute("statementType", StatementType.PREPARED.toString()));
            ResultSetType resultSetTypeEnum = this.resolveResultSetType(resultSetType);
            String nodeName = this.context.getNode().getNodeName();
            SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
            boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
            boolean flushCache = this.context.getBooleanAttribute("flushCache", !isSelect);
            boolean useCache = this.context.getBooleanAttribute("useCache", isSelect);
            boolean resultOrdered = this.context.getBooleanAttribute("resultOrdered", false);
            XMLIncludeTransformer includeParser = new XMLIncludeTransformer(this.configuration, this.builderAssistant);
            includeParser.applyIncludes(this.context.getNode());
            this.processSelectKeyNodes(id, parameterTypeClass, langDriver);
            SqlSource sqlSource = langDriver.createSqlSource(this.configuration, this.context, parameterTypeClass);
            String resultSets = this.context.getStringAttribute("resultSets");
            String keyProperty = this.context.getStringAttribute("keyProperty");
            String keyColumn = this.context.getStringAttribute("keyColumn");
            String keyStatementId = id + "!selectKey";
            keyStatementId = this.builderAssistant.applyCurrentNamespace(keyStatementId, true);
            Object keyGenerator;
            if (this.configuration.hasKeyGenerator(keyStatementId)) {
                keyGenerator = this.configuration.getKeyGenerator(keyStatementId);
            } else {
                keyGenerator = this.context.getBooleanAttribute("useGeneratedKeys", this.configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType)) ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
            }

            this.builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType, fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass, resultSetTypeEnum, flushCache, useCache, resultOrdered, (KeyGenerator)keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
        }
    }


    // 添加到configuration
    public MappedStatement addMappedStatement(String id, SqlSource sqlSource, StatementType statementType, SqlCommandType sqlCommandType, Integer fetchSize, Integer timeout, String parameterMap, Class<?> parameterType, String resultMap, Class<?> resultType, ResultSetType resultSetType, boolean flushCache, boolean useCache, boolean resultOrdered, KeyGenerator keyGenerator, String keyProperty, String keyColumn, String databaseId, LanguageDriver lang, String resultSets) {
        if (this.unresolvedCacheRef) {
            throw new IncompleteElementException("Cache-ref not yet resolved");
        } else {
            id = this.applyCurrentNamespace(id, false);
            boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
            org.apache.ibatis.mapping.MappedStatement.Builder statementBuilder = (new org.apache.ibatis.mapping.MappedStatement.Builder(this.configuration, id, sqlSource, sqlCommandType)).resource(this.resource).fetchSize(fetchSize).timeout(timeout).statementType(statementType).keyGenerator(keyGenerator).keyProperty(keyProperty).keyColumn(keyColumn).databaseId(databaseId).lang(lang).resultOrdered(resultOrdered).resultSets(resultSets).resultMaps(this.getStatementResultMaps(resultMap, resultType, id)).resultSetType(resultSetType).flushCacheRequired((Boolean)this.valueOrDefault(flushCache, !isSelect)).useCache((Boolean)this.valueOrDefault(useCache, isSelect)).cache(this.currentCache);
            ParameterMap statementParameterMap = this.getStatementParameterMap(parameterMap, parameterType, id);
            if (statementParameterMap != null) {
                statementBuilder.parameterMap(statementParameterMap);
            }

            MappedStatement statement = statementBuilder.build();
            this.configuration.addMappedStatement(statement);
            return statement;
        }
    }
```
# 参考

1. [mybatis官网][mybatis官网]

2. [项目代码][代码]


[mybatis官网]: http://www.mybatis.org/mybatis-3/zh/configuration.html

[代码]:https://github.com/yuanwenjian/mybatis-code
[mybatis内置别名]:http://www.mybatis.org/mybatis-3/zh/configuration.html#typeAliases