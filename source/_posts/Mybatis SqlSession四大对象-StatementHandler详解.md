---
title: Mybatis SqlSession 四大对象 StatementHandler 详解
description:  Mybatis通过SqlSession进行sql的执行,sql执行过程中涉及四种对象,及Executor,StatementHandler,ParameterHandler,ResultHandler.其中ParameterHandler处理sql参数,StatementHandler调用数据库statement执行,ResultHandler 对返回值封装,Executor mybatis执行器,统一对另三个对象就行调度
date: 2019-08-05 19:10:00
comments: true
tags: 
    - Java  
    - Mybatis
categories:
    - Mybatis
    - Java
---

mybais版本为3.4.6

StatementHandler 是SqlSession四大对象之一,通过statement执行数据库,mybatis分为三种StatmentHandler分别为

SimpleStatementHandler 简单StatementHandler与JDBC中Statement接口
PrepareStatementHandler 对应JDBC中的PreparedStatement ,预处理sql接口
CallableStatementHandler 对应JDBC中CallableStatement,处理存储过程接口

RoutingStatementHandler 这个没有实际操作,负责三个类的创建及调用
类图如下
![statementHandler][statementHandler]

StatementHandler接口各个方法及作用

```java
public interface StatementHandler {

	//从Connection中获取Stament对象
	Statement prepare(Connection connection) throws SQLException;
	
	//设置预处理参数
	void parameterize(Statement statement) throws SQLException;
	
	//调用批量操作
	void batch(Statement statement) throws SQLException;
	
	//更新操作
	int update(Statement statement) throws SQLException;
	
	//查询操作
	<E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException;
	
	//获取执行SQL语句的封装类BoundSql
	BoundSql getBoundSql();
	
	//参数处理器
	ParameterHandler getParameterHandler();

}
```
# statementHandler配置

```xml
  <insert id="insert" statementType="PREPARED" parameterType="com.yuanwj.demo.model.PhoneImei">
    
  </insert>
```
statementType有三个可选值STATEMENT,PREPARED,CALLABLE 分别对应SimpleStatementHandler,PrepareStatementHandler,CallableStatementHandler三种类型
```java
public enum StatementType {
    STATEMENT,
    PREPARED,
    CALLABLE;

    private StatementType() {
    }
}
```

# Statement 创建及使用配置

在上午分析Executor执行时,知道Executor是通过StatementHandler进行sql调用
```java
    //DefaultSqlSession#selectList 从configuration获取MappedStatement
    public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
        List var5;
        try {
            //获取MappedStatement
            MappedStatement ms = this.configuration.getMappedStatement(statement);
            //调用executor执行查询
            var5 = this.executor.query(ms, this.wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
        } catch (Exception var9) {
            throw ExceptionFactory.wrapException("Error querying database.  Cause: " + var9, var9);
        } finally {
            ErrorContext.instance().reset();
        }

        return var5;
    }

    //SimpleExecuor#doQuery()
    public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
        Statement stmt = null;

        List var9;
        try {
            Configuration configuration = ms.getConfiguration(); //获取configuration
            // configuration创建StatementHandler
            StatementHandler handler = configuration.newStatementHandler(this.wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
            // 通过StatemetnHandler 创建Statement
            stmt = this.prepareStatement(handler, ms.getStatementLog());
            // 执行查询
            var9 = handler.query(stmt, resultHandler);
        } finally {
            this.closeStatement(stmt);
        }
        return var9;
    }
    // Configuration#newStatementHandler configuration通过MappedStatement创建StatementHandler
    public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
        StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
        StatementHandler statementHandler = (StatementHandler)this.interceptorChain.pluginAll(statementHandler);
        return statementHandler;
    }

    // RoutingStatementHandler 构造方法 ,通过MappedStatement中StatementType 创建对应的StatementHandler
    public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
        switch(ms.getStatementType()) {
        case STATEMENT:
            this.delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        case PREPARED:
            this.delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        case CALLABLE:
            this.delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        default:
            throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
        }
    }

```
上面分析StatementHandler 创建是根据MappedStatment的类型,下面分析MappedStatment是如何通过配置创建的
```java
    // XMLMapperBuilder#configurationElement() 解析mapper文件
    private void configurationElement(XNode context) {
        try {
            String namespace = context.getStringAttribute("namespace");
            if (namespace != null && !namespace.equals("")) {
                this.builderAssistant.setCurrentNamespace(namespace);
                this.cacheRefElement(context.evalNode("cache-ref"));
                this.cacheElement(context.evalNode("cache"));
                this.parameterMapElement(context.evalNodes("/mapper/parameterMap"));
                this.resultMapElements(context.evalNodes("/mapper/resultMap"));
                this.sqlElement(context.evalNodes("/mapper/sql"));
                // 解析select,update 对应sql创建MappedStatement
                this.buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
            } else {
                throw new BuilderException("Mapper's namespace cannot be empty");
            }
        } catch (Exception var3) {
            throw new BuilderException("Error parsing Mapper XML. The XML location is '" + this.resource + "'. Cause: " + var3, var3);
        }
    }

    // XMLMapperBuilder#buildStatementFromContext()创建MappedStatement放入configuration中,MappedStatement是在
    private void buildStatementFromContext(List<XNode> list) {
        if (this.configuration.getDatabaseId() != null) {
            this.buildStatementFromContext(list, this.configuration.getDatabaseId());
        }
        this.buildStatementFromContext(list, (String)null);
    }
    // XMLMapperBuilder#buildStatementFromContext() 创建StatementHandler
    private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
        Iterator var3 = list.iterator();

        while(var3.hasNext()) {
            XNode context = (XNode)var3.next();
            XMLStatementBuilder statementParser = new XMLStatementBuilder(this.configuration, this.builderAssistant, context, requiredDatabaseId);

            try {
                //解析mapper节点各个参数放入configuration中mappedStatements ,mappedStatements为id为key,MappedStatement为值得map
                statementParser.parseStatementNode();
            } catch (IncompleteElementException var7) {
                //出现异常放入configuration中incompleteStatements参数,后续再解析
                this.configuration.addIncompleteStatement(statementParser);
            }
        }
    }

    //XMLStatementBuilder#parseStatementNode() 解析mapper节点 各个参数 生成MappedStatemend
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
            // 获取StatementType 默认为PREPARED
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
    //MapperBuilderAssistant#addMappedStatement()添加到configuration中
    public MappedStatement addMappedStatement(String id, SqlSource sqlSource, StatementType statementType, SqlCommandType sqlCommandType, Integer fetchSize, Integer timeout, String parameterMap, Class<?> parameterType, String resultMap, Class<?> resultType, ResultSetType resultSetType, boolean flushCache, boolean useCache, boolean resultOrdered, KeyGenerator keyGenerator, String keyProperty, String keyColumn, String databaseId, LanguageDriver lang, String resultSets) {
        if (this.unresolvedCacheRef) {
            throw new IncompleteElementException("Cache-ref not yet resolved");
        } else {
            id = this.applyCurrentNamespace(id, false);
            boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
            //通过构造器模式创建MappedStatement 
            org.apache.ibatis.mapping.MappedStatement.Builder statementBuilder = (new org.apache.ibatis.mapping.MappedStatement.Builder(this.configuration, id, sqlSource, sqlCommandType)).resource(this.resource).fetchSize(fetchSize).timeout(timeout).statementType(statementType).keyGenerator(keyGenerator).keyProperty(keyProperty).keyColumn(keyColumn).databaseId(databaseId).lang(lang).resultOrdered(resultOrdered).resultSets(resultSets).resultMaps(this.getStatementResultMaps(resultMap, resultType, id)).resultSetType(resultSetType).flushCacheRequired((Boolean)this.valueOrDefault(flushCache, !isSelect)).useCache((Boolean)this.valueOrDefault(useCache, isSelect)).cache(this.currentCache);
            ParameterMap statementParameterMap = this.getStatementParameterMap(parameterMap, parameterType, id);
            if (statementParameterMap != null) {
                statementBuilder.parameterMap(statementParameterMap);
            }
            // 创建MappedStatement
            MappedStatement statement = statementBuilder.build();
            // 添加到configuration中
            this.configuration.addMappedStatement(statement);
            return statement;
        }
    }

    // Configuration#addMappedStatement()
    public void addMappedStatement(MappedStatement ms) {
        this.mappedStatements.put(ms.getId(), ms);
    }
```

# StatementHandler区别
各个handler区别在于
```java
    //SimpleExecutor#prepareStatement() executor创建Statement
    private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
        Connection connection = this.getConnection(statementLog);
        Statement stmt = handler.prepare(connection, this.transaction.getTimeout()); //创建Statement
        handler.parameterize(stmt);
        return stmt;
    }
    //BaseStatementHandler#prepare() 创建Statement
    public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
        ErrorContext.instance().sql(this.boundSql.getSql());
        Statement statement = null;

        try {
            statement = this.instantiateStatement(connection);//调用各个handler创建自己的Statement
            this.setStatementTimeout(statement, transactionTimeout);
            this.setFetchSize(statement);
            return statement;
        } catch (SQLException var5) {
            this.closeStatement(statement);
            throw var5;
        } catch (Exception var6) {
            this.closeStatement(statement);
            throw new ExecutorException("Error preparing statement.  Cause: " + var6, var6);
        }
    }
```

SimpleStatementHandler ,调用connection.createStatement()返回类型为Statement
``` java
   protected Statement instantiateStatement(Connection connection) throws SQLException {
        return this.mappedStatement.getResultSetType() != null ? connection.createStatement(this.mappedStatement.getResultSetType().getValue(), 1007) : connection.createStatement(); //返回类型类型为Statement
    }
```
PreparedStatementHandler 调用connection.prepareStatement 返回PreparedStatement类型
```java
    protected Statement instantiateStatement(Connection connection) throws SQLException {
        String sql = this.boundSql.getSql();
        if (this.mappedStatement.getKeyGenerator() instanceof Jdbc3KeyGenerator) {
            String[] keyColumnNames = this.mappedStatement.getKeyColumns();
            return keyColumnNames == null ? connection.prepareStatement(sql, 1) : connection.prepareStatement(sql, keyColumnNames);
        } else {
            //返回类型为PreparedStatement
            return this.mappedStatement.getResultSetType() != null ? connection.prepareStatement(sql, this.mappedStatement.getResultSetType().getValue(), 1007) : connection.prepareStatement(sql);
        }
    }

```
CallableStatementHandler 调用connection.prepareCall 返回CallableStatement
```java
    protected Statement instantiateStatement(Connection connection) throws SQLException {
        String sql = this.boundSql.getSql();
        //返回类型为CallableStatement
        return this.mappedStatement.getResultSetType() != null ? connection.prepareCall(sql, this.mappedStatement.getResultSetType().getValue(), 1007) : connection.prepareCall(sql);
    }
```

# Mybatis StatementHandler 配置总结
1. 在mapper中配置statementType 默认为PREPARED
2. 解析mapper时,解析SELECT,UPDATE等语句时,创建MethodStatemnt的对象,methodStatemnt对象statementType类型为配置类型,
(出现异常放入configuration incompleteStatements)
3. 将上步创建的MethodStatemnt,以namespace+id为key,放入configuration中mappedStatements 类型为Map到的属性中
4. sqlSession 执行sql,解析sql时获取 namespace+id,从configuration获取对应的MethodStatemnt
(如果map中不存在,重新解析一边configuration中incompleteStatements)
5. executor执行时sql时,configuration 通过MethodStatemnt中StatementType类型创建不同类型的StateMentHandler
6. 通过StateMentHandler 创建不同的Statement类型对象,执行sql
7. statement执行,完成

[statementHandler]:../images/mybatis/statementHandler.png