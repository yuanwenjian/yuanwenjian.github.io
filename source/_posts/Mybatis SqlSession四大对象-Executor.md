---
title: Mybatis Executor 详解
description:  Mybatis Executor 详解
date: 2019-08-04 16:31:00
comments: true
tags: 
    - Java  
    - Mybatis
categories:
    - Mybatis
    - Java
---

Mybatis通过SqlSession进行sql的执行,sql执行过程中涉及四种对象,及Executor,StatementHandler,ParameterHandler,ResultHandler.其中
ParameterHandler处理sql参数,StatementHandler调用数据库statement执行,ResultHandler 对返回值封装,Executor mybatis执行器,统一对另三个对象就行调度

# Executor 配置
```xml
    <setting name="defaultExecutorType" value="SIMPLE"/>
```
ExecutorType 有三种类型 分别为
SIMPLE, SimpleExecutor 简单执行器,每次执行都会新建Statement 
REUSE,ReuseExecutor 重复利用之前创建好的Statement的对象,如果sql语句相同的话
BATCH,BatchExecutor 批量执行所有更新语句
# 创建Configuration 与SqlSession 源码分析
程序中创建Sqlsession并执行查询示例,
```java 
    
    public static void init() {
        String resource = "mybatis-config.xml";
        try {
            Reader reader = Resources.getResourceAsReader(resource);
            SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder(); //创建SqlSessionFactoryBuilder
            SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(reader); //通过SqlSessionFactoryBuilder创建SqlSessionFactory
            SqlSession sqlSession = sqlSessionFactory.openSession(); //创建SqlSession对象
            UserMapper mapper = sqlSession.getMapper(UserMapper.class);//获取mapper
            List<User> users = mapper.selectAll();//执行查询
            System.out.println(users.size());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
创建SqlSessionFactoryBuilder源码,最终默认返回为DefaultSqlSessionFactory
```java
    // SqlSessionFactoryBuilder#build() 
    public SqlSessionFactory build(Reader reader) {
        return this.build((Reader)reader, (String)null, (Properties)null);
    }

    //创建SqlSessionFactory方法 
    public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
        SqlSessionFactory var5;
        try {
            XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties); //创建XMLConfigBuilder,并创建documnet对象,用于下一步解析
            var5 = this.build(parser.parse()); //解析配置,获取mapper,sql,datasource,properties,settings,typeAliases等节点内容
            // 其中executor type在这一步解析,并最终设置到configuration与SqlSession 中
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

    //创建SqlSessionFactory,并将解析结果添加到SqlSessionFactory
     public SqlSessionFactory build(Configuration config) {
        return new DefaultSqlSessionFactory(config);
    }
```

创建Sqlsession源码
```java
    //SqlSessionFactory创建SqlSession 方法,默认返回DefaultSqlSession
    public SqlSession openSession() {
        return this.openSessionFromDataSource(this.configuration.getDefaultExecutorType(), (TransactionIsolationLevel)null, false);
    }

    private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
        Transaction tx = null;

        DefaultSqlSession var8;
        try {
            Environment environment = this.configuration.getEnvironment();
            TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(environment);
            tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
            Executor executor = this.configuration.newExecutor(tx, execType); //根据execType创建对应的Executor,后面最终由她执行sql
            var8 = new DefaultSqlSession(this.configuration, executor, autoCommit);
        } catch (Exception var12) {
            this.closeTransaction(tx);
            throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
        } finally {
            ErrorContext.instance().reset();
        }

        return var8;
    }
```

# Executor配置解析并关联到Sqlssion 并在查询中使用中
上面分析创建SqlSessionFactory与SqlSession 的过程,那下面分析配置的defaultExecutorType如何添加到SqlSession中
```java 
// ExecutorType 枚举,执行时判断使用哪个Executor
public enum ExecutorType {
    SIMPLE,
    REUSE,
    BATCH;

    private ExecutorType() {
    }
}
    //XMLConfigBuilder#parseConfiguration() 解析配置文件
    private void parseConfiguration(XNode root) {
        try {
            this.propertiesElement(root.evalNode("properties"));
            Properties settings = this.settingsAsProperties(root.evalNode("settings"));
            this.loadCustomVfs(settings);
            this.typeAliasesElement(root.evalNode("typeAliases"));
            this.pluginElement(root.evalNode("plugins"));
            this.objectFactoryElement(root.evalNode("objectFactory"));
            this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
            this.reflectorFactoryElement(root.evalNode("reflectorFactory"));
            this.settingsElement(settings); //将settings节点下文件配置到configuration 中
            this.environmentsElement(root.evalNode("environments"));
            this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
            this.typeHandlerElement(root.evalNode("typeHandlers"));
            this.mapperElement(root.evalNode("mappers"));
        } catch (Exception var3) {
            throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
        }
    }
    //XMLConfigBuilder#settingsElement() 将配置添加到configuration中 处于解析xml环节中
    private void settingsElement(Properties props) throws Exception {
        this.configuration.setAutoMappingBehavior(AutoMappingBehavior.valueOf(props.getProperty("autoMappingBehavior", "PARTIAL")));
        this.configuration.setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior.valueOf(props.getProperty("autoMappingUnknownColumnBehavior", "NONE")));
        this.configuration.setCacheEnabled(this.booleanValueOf(props.getProperty("cacheEnabled"), true));
        this.configuration.setProxyFactory((ProxyFactory)this.createInstance(props.getProperty("proxyFactory")));
        this.configuration.setLazyLoadingEnabled(this.booleanValueOf(props.getProperty("lazyLoadingEnabled"), false));
        this.configuration.setAggressiveLazyLoading(this.booleanValueOf(props.getProperty("aggressiveLazyLoading"), false));
        this.configuration.setMultipleResultSetsEnabled(this.booleanValueOf(props.getProperty("multipleResultSetsEnabled"), true));
        this.configuration.setUseColumnLabel(this.booleanValueOf(props.getProperty("useColumnLabel"), true));
        this.configuration.setUseGeneratedKeys(this.booleanValueOf(props.getProperty("useGeneratedKeys"), false));
        this.configuration.setDefaultExecutorType(ExecutorType.valueOf(props.getProperty("defaultExecutorType", "SIMPLE"))); //读取defaultExecutorType配置,默认为SIMPLE,设置到configuration中
        this.configuration.setDefaultStatementTimeout(this.integerValueOf(props.getProperty("defaultStatementTimeout"), (Integer)null));
        // ....
    }    

    // SqlSessionFactory#openSession() 创建SqlSession
    public SqlSession openSession() {
        return this.openSessionFromDataSource(this.configuration.getDefaultExecutorType(), (TransactionIsolationLevel)null, false);
    }
    //创建SqlSession
    private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
        Transaction tx = null;

        DefaultSqlSession var8;
        try {
            Environment environment = this.configuration.getEnvironment();
            TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(environment);
            tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
            Executor executor = this.configuration.newExecutor(tx, execType); //创建Executor
            var8 = new DefaultSqlSession(this.configuration, executor, autoCommit);
        } catch (Exception var12) {
            this.closeTransaction(tx);
            throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
        } finally {
            ErrorContext.instance().reset();
        }
        return var8;
    }

    // 创建Executor
    public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
        executorType = executorType == null ? this.defaultExecutorType : executorType;
        executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
        Object executor;
        if (ExecutorType.BATCH == executorType) {
            executor = new BatchExecutor(this, transaction);
        } else if (ExecutorType.REUSE == executorType) {
            executor = new ReuseExecutor(this, transaction);
        } else {
            executor = new SimpleExecutor(this, transaction);
        }

        if (this.cacheEnabled) {
            executor = new CachingExecutor((Executor)executor);
        }

        Executor executor = (Executor)this.interceptorChain.pluginAll(executor);
        return executor;
    }
```
上面已经将executor配置到SqlSession中,下面是使用代码,使用mapper查询时,mapper为动态代理创建,MapperProxy#invoke()监控mapper所有执行方法,及所有方法最中调用MapperMethod#execute()方法,
```java
    //MapperProxy#invoke()
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
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

        MapperMethod mapperMethod = this.cachedMapperMethod(method);
        return mapperMethod.execute(this.sqlSession, args);
    }

    // MapperMethod#execute() 这一步为解析sql,查询,返回结果
    public Object execute(SqlSession sqlSession, Object[] args) {
        Object param;
        Object result;
        switch(this.command.getType()) {
        case INSERT: //插入
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.insert(this.command.getName(), param));
            break;
        case UPDATE: //更新
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.update(this.command.getName(), param));
            break;
        case DELETE: //删除
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.delete(this.command.getName(), param));
            break;
        case SELECT://查询
            if (this.method.returnsVoid() && this.method.hasResultHandler()) {
                this.executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (this.method.returnsMany()) {
                result = this.executeForMany(sqlSession, args);//返回list,多条数据
            } else if (this.method.returnsMap()) {
                result = this.executeForMap(sqlSession, args);
            } else if (this.method.returnsCursor()) {
                result = this.executeForCursor(sqlSession, args);
            } else {
                param = this.method.convertArgsToSqlCommandParam(args);
                result = sqlSession.selectOne(this.command.getName(), param); //返回单条
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

上面得知所有执行sql 都是根据sqlSession 执行了,并且SqlSession 默认为DefaultSqlSession .下面以update为例,分析Executor使用

```java
    //DefaultSqlSession#update() 
    public int update(String statement, Object parameter) {
        int var4;
        try {
            this.dirty = true;
            MappedStatement ms = this.configuration.getMappedStatement(statement);
            var4 = this.executor.update(ms, this.wrapCollection(parameter)); //执行update
        } catch (Exception var8) {
            throw ExceptionFactory.wrapException("Error updating database.  Cause: " + var8, var8);
        } finally {
            ErrorContext.instance().reset();
        }

        return var4;
    }

```
var4 = this.executor.update(ms, this.wrapCollection(parameter));
上文中总算找到Executor方法了,当前executor 为配置configuration中配置的类型及BaseExecutor,ReuseExecutor,BatchExecutor类型之一.下面分析这三种区别.

最终sql执行通过Executor doQuery()和doUpdate()方法,以上文update()为例
BaseExecutor#doUpdate() 每次返回新的Statement
```java
    public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
        Statement stmt = null;

        int var6;
        try {
            Configuration configuration = ms.getConfiguration();
            StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, (ResultHandler)null, (BoundSql)null);
            stmt = this.prepareStatement(handler, ms.getStatementLog());
            var6 = handler.update(stmt); ////调用updaet方法
        } finally {
            this.closeStatement(stmt);
        }

        return var6;
    }
    // BaseExecutor#prepareStatement()每次都返回新的Statement
    private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
        Connection connection = this.getConnection(statementLog);
        Statement stmt = handler.prepare(connection, this.transaction.getTimeout());
        handler.parameterize(stmt);
        return stmt;
    }
```
ReuseExecuor#doQuery()方法,复用之前Statement对象
```java

    public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
        Configuration configuration = ms.getConfiguration();
        StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, (ResultHandler)null, (BoundSql)null);
        Statement stmt = this.prepareStatement(handler, ms.getStatementLog());//复用之前的
        return handler.update(stmt); //调用updaet方法
    }
    //ReuseExecuor#prepareStatement()
    private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
        BoundSql boundSql = handler.getBoundSql();
        String sql = boundSql.getSql();
        Statement stmt;
        if (this.hasStatementFor(sql)) { //如果存在复用
            stmt = this.getStatement(sql);
            this.applyTransactionTimeout(stmt);
        } else { //不存在新建,并将新建的放入statementMap中
            Connection connection = this.getConnection(statementLog);
            stmt = handler.prepare(connection, this.transaction.getTimeout());
            this.putStatement(sql, stmt);
        }

        handler.parameterize(stmt);
        return stmt;
    }

    //ReuseExecuor#hasStatementFor()判断之前使用存在,并且连接没有关闭
    private boolean hasStatementFor(String sql) {
        try {
            return this.statementMap.keySet().contains(sql) && !((Statement)this.statementMap.get(sql)).getConnection().isClosed();
        } catch (SQLException var3) {
            return false;
        }
    }
    //放入statementMap中
    private void putStatement(String sql, Statement stmt) {
        this.statementMap.put(sql, stmt);
    }
    // statementMap为sql为key,Statement为值的map 
    Map<String, Statement> statementMap = new HashMap();
```
BatchExecutor#doUpdate()
```java

    public int doUpdate(MappedStatement ms, Object parameterObject) throws SQLException {
        Configuration configuration = ms.getConfiguration();
        StatementHandler handler = configuration.newStatementHandler(this, ms, parameterObject, RowBounds.DEFAULT, (ResultHandler)null, (BoundSql)null);
        BoundSql boundSql = handler.getBoundSql();
        String sql = boundSql.getSql();
        Statement stmt;
        if (sql.equals(this.currentSql) && ms.equals(this.currentStatement)) {//如果sql相同并且statement也相同
            int last = this.statementList.size() - 1;
            stmt = (Statement)this.statementList.get(last);
            this.applyTransactionTimeout(stmt);
            handler.parameterize(stmt);
            BatchResult batchResult = (BatchResult)this.batchResultList.get(last);
            batchResult.addParameterObject(parameterObject);
        } else {//新建
            Connection connection = this.getConnection(ms.getStatementLog());
            stmt = handler.prepare(connection, this.transaction.getTimeout());
            handler.parameterize(stmt);
            this.currentSql = sql;
            this.currentStatement = ms;
            this.statementList.add(stmt);
            this.batchResultList.add(new BatchResult(ms, sql, parameterObject));
        }

        handler.batch(stmt);//执行batch方法
        return -2147482646;
    }
```