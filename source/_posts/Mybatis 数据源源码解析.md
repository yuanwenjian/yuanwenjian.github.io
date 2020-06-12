---
title: Mybatis DataSource数据源实现源码与解析
description:  Mybatis 内置三种数据源即UnpooledDataSource,PooledDataSource,JndiDataSource,UnpooledDataSource 不使用连接池每次新建连接,PooledDataSource使用连接池,JndiDataSource使用容器连接池,本篇不考虑,类图如下
date: 2019-08-04 16:31:00
comments: true
tags: 
    - Java  
    - Mybatis
categories:
    - Mybatis
    - Java
---
![datasource1][datasource1]

# 两个数据源工厂类

## UnpooledDataSourceFactory
```java

public class UnpooledDataSourceFactory implements DataSourceFactory {
    private static final String DRIVER_PROPERTY_PREFIX = "driver.";
    private static final int DRIVER_PROPERTY_PREFIX_LENGTH = "driver.".length();
    protected DataSource dataSource = new UnpooledDataSource();

    public UnpooledDataSourceFactory() {
    }

    public void setProperties(Properties properties) {
        Properties driverProperties = new Properties();
        //通过反射获取配置信息
        MetaObject metaDataSource = SystemMetaObject.forObject(this.dataSource);
        Iterator var4 = properties.keySet().iterator();

        while(var4.hasNext()) {
            Object key = var4.next();
            String propertyName = (String)key;
            String value;
            if (propertyName.startsWith("driver.")) {
                //设置驱动类型
                value = properties.getProperty(propertyName);
                driverProperties.setProperty(propertyName.substring(DRIVER_PROPERTY_PREFIX_LENGTH), value);
            } else {
                //通过属性设置值,如果属性不存在报错
                if (!metaDataSource.hasSetter(propertyName)) {
                    throw new DataSourceException("Unknown DataSource property: " + propertyName);
                }

                value = (String)properties.get(propertyName);
                //获取属性值
                Object convertedValue = this.convertValue(metaDataSource, propertyName, value);
                metaDataSource.setValue(propertyName, convertedValue);
            }
        }

        if (driverProperties.size() > 0) {
            metaDataSource.setValue("driverProperties", driverProperties);
        }

    }

    public DataSource getDataSource() {
        return this.dataSource;
    }

    private Object convertValue(MetaObject metaDataSource, String propertyName, String value) {
        Object convertedValue = value;
        Class<?> targetType = metaDataSource.getSetterType(propertyName);
        if (targetType != Integer.class && targetType != Integer.TYPE) {
            if (targetType != Long.class && targetType != Long.TYPE) {
                if (targetType == Boolean.class || targetType == Boolean.TYPE) {
                    convertedValue = Boolean.valueOf(value);
                }
            } else {
                convertedValue = Long.valueOf(value);
            }
        } else {
            convertedValue = Integer.valueOf(value);
        }

        return convertedValue;
    }
}
```

## PooledDataSourceFactory
```java
public class PooledDataSourceFactory extends UnpooledDataSourceFactory {
    public PooledDataSourceFactory() {
        this.dataSource = new PooledDataSource();
    }
} 
```
PooledDataSourceFactory 继承UnpooledDataSourceFactory 只是返回数据源为PooledDataSource,其他属性配置不变

## UnpooledDataSource 获取连接
```java

    private Connection doGetConnection(Properties properties) throws SQLException {
        //初始化驱动
        this.initializeDriver();
        //获取连接
        Connection connection = DriverManager.getConnection(this.url, properties);
        //配置自动提交与事务隔离级别
        this.configureConnection(connection);
        return connection;
    }

    private synchronized void initializeDriver() throws SQLException {
        if (!registeredDrivers.containsKey(this.driver)) {
            try {
                Class driverType;
                if (this.driverClassLoader != null) {
                    driverType = Class.forName(this.driver, true, this.driverClassLoader);
                } else {
                    driverType = Resources.classForName(this.driver);
                }
                //创建驱动
                Driver driverInstance = (Driver)driverType.newInstance();
                //注册驱动
                DriverManager.registerDriver(new UnpooledDataSource.DriverProxy(driverInstance));
                //添加属性registeredDrivers,registeredDrivers类型为ConcurrentHashMap能保证线程安全
                registeredDrivers.put(this.driver, driverInstance);
            } catch (Exception var3) {
                throw new SQLException("Error setting driver on UnpooledDataSource. Cause: " + var3);
            }
        }

    }

    private void configureConnection(Connection conn) throws SQLException {
        if (this.autoCommit != null && this.autoCommit != conn.getAutoCommit()) {
            //设置自动提交
            conn.setAutoCommit(this.autoCommit);
        }

        if (this.defaultTransactionIsolationLevel != null) {
            //设置事务隔离级别
            conn.setTransactionIsolation(this.defaultTransactionIsolationLevel);
        }
    }
```
上面就是UnpooledDataSource 获取连接逻辑,即每次会新增一连接,创建连接通常会很耗时,并且连接资源很宝贵,所以连接需要交给连接池管理,Mybatis也有连接池的实现即PooledDataSource

## PooledDataSource

### PooledDataSource,PoolState,PooledConnection三者关系为下图
![datasource2][datasource2]

PooledDataSource 由UnpooledDataSource创建,并且自身不维护连接,只管理PooledConnection,但是由于是连接池所以还需要维护连接池大小,状态,可用连接等,所以有PoolState

### PooledConnection
它主要是用来管理数据库连接的，它是一个代理类，实现了 InvocationHandler 接口,由于实现InvocationHandler,查看invoke方法
```java
class PooledConnection implements InvocationHandler {
    private static final String CLOSE = "close";
    private static final Class<?>[] IFACES = new Class[]{Connection.class};
    private final int hashCode;
    private final PooledDataSource dataSource;
    private final Connection realConnection; //真正的数据库连接
    private final Connection proxyConnection; //代理对象
    private long checkoutTimestamp; //最后一次取出时间
    private long createdTimestamp;//创建实际戳
    private long lastUsedTimestamp;//最后使用时间戳
    private int connectionTypeCode;
    private boolean valid;//是否有效

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        //如果是关闭连接则会放回连接池
        if ("close".hashCode() == methodName.hashCode() && "close".equals(methodName)) {
            this.dataSource.pushConnection(this);
            return null;
        } else {
            try {
                if (!Object.class.equals(method.getDeclaringClass())) {
                    this.checkConnection();
                }

                return method.invoke(this.realConnection, args);
            } catch (Throwable var6) {
                throw ExceptionUtil.unwrapThrowable(var6);
            }
        }
    }
}

```

### PoolState

PoolState 类主要是用来管理连接池的状态，比如哪些连接是空闲的，哪些是活动的
```java
public class PoolState {
    protected PooledDataSource dataSource;
    //空闲连接
    protected final List<PooledConnection> idleConnections = new ArrayList(); 
    //活动连接
    protected final List<PooledConnection> activeConnections = new ArrayList();
    //请求数据库连接的次数
    protected long requestCount = 0L;
    //累计时间
    protected long accumulatedRequestTime = 0L;
    protected long accumulatedCheckoutTime = 0L;
    protected long claimedOverdueConnectionCount = 0L;
    protected long accumulatedCheckoutTimeOfOverdueConnections = 0L;
    protected long accumulatedWaitTime = 0L;
    protected long hadToWaitCount = 0L;
    protected long badConnectionCount = 0L;
}
```

### PooledDataSource 获取连接
```java
    public Connection getConnection() throws SQLException {
        return this.popConnection(this.dataSource.getUsername(), this.dataSource.getPassword()).getProxyConnection();
    }

    private PooledConnection popConnection(String username, String password) throws SQLException {
        boolean countedWait = false;
        PooledConnection conn = null;
        long t = System.currentTimeMillis();
        int localBadConnectionCount = 0;

        while(conn == null) {
            PoolState var8 = this.state;
            synchronized(this.state) {
                if (!this.state.idleConnections.isEmpty()) {
                    conn = (PooledConnection)this.state.idleConnections.remove(0);
                    if (log.isDebugEnabled()) {
                        log.debug("Checked out connection " + conn.getRealHashCode() + " from pool.");
                    }
                } else if (this.state.activeConnections.size() < this.poolMaximumActiveConnections) {
                    conn = new PooledConnection(this.dataSource.getConnection(), this);
                    if (log.isDebugEnabled()) {
                        log.debug("Created connection " + conn.getRealHashCode() + ".");
                    }
                } else {
                    PooledConnection oldestActiveConnection = (PooledConnection)this.state.activeConnections.get(0);
                    long longestCheckoutTime = oldestActiveConnection.getCheckoutTime();
                    if (longestCheckoutTime > (long)this.poolMaximumCheckoutTime) {
                        ++this.state.claimedOverdueConnectionCount;
                        this.state.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;
                        this.state.accumulatedCheckoutTime += longestCheckoutTime;
                        this.state.activeConnections.remove(oldestActiveConnection);
                        if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {
                            try {
                                oldestActiveConnection.getRealConnection().rollback();
                            } catch (SQLException var16) {
                                log.debug("Bad connection. Could not roll back");
                            }
                        }

                        conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);
                        conn.setCreatedTimestamp(oldestActiveConnection.getCreatedTimestamp());
                        conn.setLastUsedTimestamp(oldestActiveConnection.getLastUsedTimestamp());
                        oldestActiveConnection.invalidate();
                        if (log.isDebugEnabled()) {
                            log.debug("Claimed overdue connection " + conn.getRealHashCode() + ".");
                        }
                    } else {
                        try {
                            if (!countedWait) {
                                ++this.state.hadToWaitCount;
                                countedWait = true;
                            }

                            if (log.isDebugEnabled()) {
                                log.debug("Waiting as long as " + this.poolTimeToWait + " milliseconds for connection.");
                            }

                            long wt = System.currentTimeMillis();
                            this.state.wait((long)this.poolTimeToWait);
                            this.state.accumulatedWaitTime += System.currentTimeMillis() - wt;
                        } catch (InterruptedException var17) {
                            break;
                        }
                    }
                }

                if (conn != null) {
                    if (conn.isValid()) {
                        if (!conn.getRealConnection().getAutoCommit()) {
                            conn.getRealConnection().rollback();
                        }

                        conn.setConnectionTypeCode(this.assembleConnectionTypeCode(this.dataSource.getUrl(), username, password));
                        conn.setCheckoutTimestamp(System.currentTimeMillis());
                        conn.setLastUsedTimestamp(System.currentTimeMillis());
                        this.state.activeConnections.add(conn);
                        ++this.state.requestCount;
                        this.state.accumulatedRequestTime += System.currentTimeMillis() - t;
                    } else {
                        if (log.isDebugEnabled()) {
                            log.debug("A bad connection (" + conn.getRealHashCode() + ") was returned from the pool, getting another connection.");
                        }

                        ++this.state.badConnectionCount;
                        ++localBadConnectionCount;
                        conn = null;
                        if (localBadConnectionCount > this.poolMaximumIdleConnections + this.poolMaximumLocalBadConnectionTolerance) {
                            if (log.isDebugEnabled()) {
                                log.debug("PooledDataSource: Could not get a good connection to the database.");
                            }

                            throw new SQLException("PooledDataSource: Could not get a good connection to the database.");
                        }
                    }
                }
            }
        }

        if (conn == null) {
            if (log.isDebugEnabled()) {
                log.debug("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
            }

            throw new SQLException("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
        } else {
            return conn;
        }
    }

```

首先获取的代理数据库连接PooledConnection,该对象通过维护真是Connection,并通过JDK的动态代理产生真实的数据库Connection连接对象
循环获取PooledConnection，知道获取得到为止
首先拿到连接池状态锁，判断空闲连接池是否为空，如果不为空,获取第一个空闲连接(同时remove从空闲池集合中remove第一个),并返回
如果空闲连接池为空,判断当前激活连接池大小是否小于连接池最大连接数,如果小于则new一个新的PooledConnection代理连接对象
如果连接池最大连接数已满,获取第一个连接池代理对象,判断该代理对象是否预期(连接池预期时间默认20秒),如果当前连接已预期,从活动连接池中移除该连接,回滚当前连接事务，使用当前代理对象的Connection，作为创建新的PooledConnection对象的参数，老的连接置为不可用
如果第一个池对象并未预期,则等待，根据连接池的等待时间进行等待
拿到PooledConnection对象后,对当前连接判断是否有效,有效的方法验证主要包括当前连接是否关闭，如果只想query操作,并执行,执行拿到结果则当前连接为可用连接
拿到真实Connection对象,判断是否自动提交,如果为false,则回滚当前Connection的事务，最后将该连接加入到连接池活动连接集合中返回
当我们得到PooledConnection后，因为最终要返回的是Connection对象,随意调用池代理连接的getProxyConnect()方法获取代理对象
获取代理对象时,首先判断当前Connection的方法，如果是调用close()方法,则首先会进入释放连接的逻辑，从当前活动连接池中移除该对象,判断空闲池空间足够,如果可用,加入空闲连接池,调用notifyAll(),唤醒wait线程,如果空闲连接池已满,则真实调用Connection的close()方法,并在之前回滚事务,关闭该连接


[datasource1]:/images/mybatis/datasource1.png
[datasource2]:/images/mybatis/datasource2.png


