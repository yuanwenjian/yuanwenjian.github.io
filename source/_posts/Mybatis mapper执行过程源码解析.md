---
title: Mybatis mapper执行sql源码过程分析
description:  在Mybatis开发过程中，使用sqlSession.getMapper(mapper.class)即可获取mapper接口，并可以直接调用其方法进行sql增删改查，本文分析这一步源码是怎么运行的
date: 2018-05-05 20:40:00
comments: true
tags: 
    - Java  
categories:
    - Mybatis
---

# Java方法调用mapper方法,代码如下所示
```java
    PhoneImeiMapper mapper = sqlSession.getMapper(PhoneImeiMapper.class); //通过mapper.class获取
    List<PhoneImei> phoneImeiList = mapper.selectAll(); //调用查询方法
```

# 获取maaper代理类源码分析, 以下代码只分析其主要方法，不重要的忽略，

```java 
    //DefaultSqlSession 类
    public <T> T getMapper(Class<T> type) {
        return this.configuration.getMapper(type, this); //通过configuration获取mapper
    }

    //Configuration
    public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
        return this.mapperRegistry.getMapper(type, sqlSession);//继续调用
    }

    // MapperRegistry
    public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
        MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory)this.knownMappers.get(type);//根据mapper.class从knownMappers属性获取其对应的MapperProxyFactory，具体knownMappers赋值过程可查看另一篇博客，mybatis配置源码分析 // TODO 链接
        if (mapperProxyFactory == null) {
            throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
        } else {
            try {
                return mapperProxyFactory.newInstance(sqlSession); //通mapperProxyFactory 代理工厂创建mapper,并与sqlSession关联
            } catch (Exception var5) {
                throw new BindingException("Error getting mapper instance. Cause: " + var5, var5);
            }
        }
    }

    //MapperProxyFactory 生成MapperProxy
    public T newInstance(SqlSession sqlSession) {
        MapperProxy<T> mapperProxy = new MapperProxy(sqlSession, this.mapperInterface, this.methodCache);
        return this.newInstance(mapperProxy);
    }

    //MapperProxyFactory 通过反射生成mapper,
    protected T newInstance(MapperProxy<T> mapperProxy) {
        // Proxy.newProxyInstance()静态方法作用可自行百度
        //  Proxy.newProxyInstance 第二个参数为具体mapper类接口，如何赋值，请看另一篇mybatis配置源码分析
        // Proxy.newProxyInstance第三个参数InvocationHandler为mapperProxy,及创建的代理对象方法都会执行MapperProxy.invoke()方法
        return Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
    }
```

# mapper执行语句分析
 
通过上一节，得知如何获取mapper代理类，本节分析mapper如何执行sql

上节得知通过反射Proxy.newProxyInstance()创建mapper代理类的InvocationHandler的参数为mapperProxy,该参数作用为所有代理类的方法调用必须invoke(),下面分析MapperProxy.invoke()方法
```java
    // MapperProxy.class
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // proxy实例对象，method执行方法，args，参数
        try {
            if (Object.class.equals(method.getDeclaringClass())) {
                return method.invoke(this, args);
            }

            if (this.isDefaultMethod(method)) {
                return this.invokeDefaultMethod(proxy, method, args);
            }
        } catch (Throwable var5) {
            throw ExceptionUtil.unwrapThrowable(var5);
        }

        MapperMethod mapperMethod = this.cachedMapperMethod(method); //获取MapperMethod，如果缓存中直接获取，没有创建并加入缓存
        return mapperMethod.execute(this.sqlSession, args);// mapper类所有方法最终执行execute方法
    }

    // MapperMethod.class 构造方法，为两个属性赋值，sqlCommand 中保存了一些和 SQL 相关的信息,MethodSignature 即方法签名，顾名思义，该类保存了一些和目标方法相关的信息。比如目标方法的返回类型，目标方法的参数列表信息等。
    public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
        this.command = new MapperMethod.SqlCommand(config, mapperInterface, method);
        this.method = new MapperMethod.MethodSignature(config, mapperInterface, method);
    }
        // MapperMethod.SqlCommand.class 构造方法，从configuration中获取SqlCommandType、name，name为方法名，sqlConmandatype即UNKNOWN,INSERT,UPDATE,DELETE,SELECT,FLUSH;该值初始化configuration时在解析mapper节点时赋值
        public SqlCommand(Configuration configuration, Class<?> mapperInterface, Method method) {
            String methodName = method.getName();
            Class<?> declaringClass = method.getDeclaringClass();
            MappedStatement ms = this.resolveMappedStatement(mapperInterface, methodName, declaringClass, configuration);
            if (ms == null) {
                if (method.getAnnotation(Flush.class) == null) {
                    throw new BindingException("Invalid bound statement (not found): " + mapperInterface.getName() + "." + methodName);
                }

                this.name = null;
                this.type = SqlCommandType.FLUSH;
            } else {
                this.name = ms.getId();
                this.type = ms.getSqlCommandType();
                if (this.type == SqlCommandType.UNKNOWN) {
                    throw new BindingException("Unknown execution method for: " + this.name);
                }
            }

        }
        // MapperMethod.MethodSignature 方法签名
        public MethodSignature(Configuration configuration, Class<?> mapperInterface, Method method) {
            Type resolvedReturnType = TypeParameterResolver.resolveReturnType(method, mapperInterface);
            if (resolvedReturnType instanceof Class) {
                this.returnType = (Class)resolvedReturnType;
            } else if (resolvedReturnType instanceof ParameterizedType) {
                this.returnType = (Class)((ParameterizedType)resolvedReturnType).getRawType();
            } else {
                this.returnType = method.getReturnType();
            }

            this.returnsVoid = Void.TYPE.equals(this.returnType);
            this.returnsMany = configuration.getObjectFactory().isCollection(this.returnType) || this.returnType.isArray();
            this.returnsCursor = Cursor.class.equals(this.returnType);
            this.mapKey = this.getMapKey(method);
            this.returnsMap = this.mapKey != null;
            this.rowBoundsIndex = this.getUniqueParamIndex(method, RowBounds.class);
            this.resultHandlerIndex = this.getUniqueParamIndex(method, ResultHandler.class);
            this.paramNameResolver = new ParamNameResolver(configuration, method);
        }

```

mapper执行sql语句

```java

    public Object execute(SqlSession sqlSession, Object[] args) {
        Object param;
        Object result;
        switch(this.command.getType()) {
        case INSERT: //对应sql类型
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.insert(this.command.getName(), param));
            break;
        case UPDATE:
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.update(this.command.getName(), param));
            break;
        case DELETE:
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.delete(this.command.getName(), param));
            break;
        case SELECT:
            if (this.method.returnsVoid() && this.method.hasResultHandler()) {
                this.executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (this.method.returnsMany()) {
                result = this.executeForMany(sqlSession, args);
            } else if (this.method.returnsMap()) {
                result = this.executeForMap(sqlSession, args);
            } else if (this.method.returnsCursor()) {
                result = this.executeForCursor(sqlSession, args);
            } else {
                param = this.method.convertArgsToSqlCommandParam(args);
                result = sqlSession.selectOne(this.command.getName(), param);
            }
            break;
        case FLUSH:
            result = sqlSession.flushStatements();
            break;
        default:
            throw new BindingException("Unknown execution method for: " + this.command.getName());
        }

        if (result == null && this.method.getReturnType().isPrimitive() && !this.method.returnsVoid()) {
            throw new BindingException("Mapper method '" + this.command.getName() + " attempted to return null from a method with a primitive return type (" + this.method.getReturnType() + ").");
        } else {
            return result;
        }
    }
```