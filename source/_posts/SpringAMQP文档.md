---
title: RabbitMQ消息确认
description: RabbitMQ是一个消息代理 - 一个消息系统的媒介。它可以为你的应用提供一个通用的消息发送和接收平台，并且保证消息在传输过程中的安全。
date: 2018-06-12 12:30:00
comments: true
tags: 
    - RabbitMQ
    - 后端  
    - Spring
categories:
    - 后端
    - Spring
---

# 文档
## AMPQ抽象

### 介绍
Spring AMQP由少数几个模块组成，每个模块在分发中都由JAR表示。这些模块是：spring-amqp和spring-rabbit。spring-amqp模块包含org.springframework.amqp.core包。在该软件包中，您将找到代表核心AMQP“模型”的类，我们的目的是提供不依赖任何特定AMQP代理实施或客户端库的通用抽象，最终用户代码在供应商实现中将更具可移植性，因为它可以仅针对抽象层进行开发。这些抽象然后由broker-specific专用模块（例如spring-rabbit）实施。目前只有RabbitMQ实现;但是除了RabbitMQ之外，抽象已经在.NET中使用Apache Qpid进行了验证。由于AMQP原则上在协议级别运行，RabbitMQ客户端可以与支持相同协议版本的任何代理一起使用，但目前我们不测试任何其他代理。

### Message
0-9-1 AMQP规范没有定义Message类或接口。但是在执行诸如basicPublish（）之类的操作时，内容将作为字节数组参数传递，而其他属性作为单独的参数传递。Spring AMQP将Message类定义为更一般的AMQP域模型表示的一部分，Message类的目的是简单地将主体和属性封装在单个实例中，以便API可以变得更简单。 Message类的定义非常简单。
```java
public class Message {

    private final MessageProperties messageProperties;

    private final byte[] body;

    public Message(byte[] body, MessageProperties messageProperties) {
        this.body = body;
        this.messageProperties = messageProperties;
    }

    public byte[] getBody() {
        return this.body;
    }

    public MessageProperties getMessageProperties() {
        return this.messageProperties;
    }
}
```

MessageProperties接口定义了几个通用属性，如messageId，timestamp，contentType等等。这些属性也可以通过调用setHeader（String key，Object value）方法用用户定义的标头进行扩展

### Exchange

Exchange接口代表AMQP Exchange，这是消息生产者发送到的东西。broker的虚拟主机中的每个交易都将具有唯一的名称以及一些其他属性
```java
public interface Exchange {

    String getName();

    String getExchangeType();

    boolean isDurable();

    boolean isAutoDelete();

    Map<String, Object> getArguments();

}
```
如您所见，Exchange还具有由ExchangeTypes中定义的常量表示的类型。基本类型有：Direct，Topic，Fanout和Headers,在核心软件包中，您可以找到每种类型的Exchange接口的实现。

### Queue

Queue类表示Message Consumer从中接收消息的组件。像各种Exchange类一样，我们的实现旨在成为此核心AMQP类型的抽象表示。

```java
public class Queue  {

    private final String name;

    private volatile boolean durable;

    private volatile boolean exclusive;

    private volatile boolean autoDelete;

    private volatile Map<String, Object> arguments;

    /**
     * The queue is durable, non-exclusive and non auto-delete.
     *
     * @param name the name of the queue.
     */
    public Queue(String name) {
        this(name, true, false, false);
    }

    // Getters and Setters omitted for brevity

}
```
请注意，构造函数接受队列名称,根据实施情况，admin template可能会提供用于生成唯一命名队列的方法。这种队列可以用作“回复”地址或其他临时情况,因此，自动生成的Queue的exclusive和autoDelete属性都将设置为true。

### Binding

考虑到生产者发送到Exchange并且消费者从队列接收到，将队列连接到Exchange的绑定对于通过消息传递连接这些生产者和消费者至关重要。在Spring AMQP中，我们定义了一个Binding类来表示这些连接。

就其本身而言，Binding类的一个实例只是保存关于连接的数据。换句话说，它不是一个“主动”组件,绑定实例可以被AmqpAdmin类用来实际触发代理上的绑定操作。另外，你会在同一节中看到，Binding实例可以在@Configuration类中使用Spring的@ Bean风格来定义。还有一个方便的基类，它进一步简化了生成与AMQP相关的bean定义并识别队列，交换和绑定的方法以便它们将在应用程序启动时在AMQP代理上声明。使用分离的连接在某些环境中非常有用，比如连接负载均衡中的不同集团成员，将cacheMode设置为CacheMode.CONNECTION。

## Connection and Resource Management 连接及资源管理

### 介绍
连接 RabbitMQ broker的核心组件是ConnectionFactory接口，ConnectionFactory实现的责任是提供一个org.springframework.amqp.rabbit.connection.Connection实例，它是com.rabbitmq.client.Connection的一个包装。我们提供的唯一具体实现是CachingConnectionFactory，默认情况下，它将建立可由应用程序共享的单个连接代理。由于使用AMQP进行消息传递的“工作单元”实际上是一个“channel”（在某些方面，这与JMS中的Connection和Session之间的关系相似），因此可以共享连接。如你所想，连接实例提供了一个createChannel方法。 CachingConnectionFactory实现支持这些通道的缓存,并根据它们是否是事务处理来维护通道的单独缓存。在创建CachingConnectionFactory的实例时，可以通过构造函数提供主机名。用户名和密码属性也应该提供。如果您想配置通道缓存的大小（默认值为25），那么您也可以在这里调用setChannelCacheSize（）方法。

从版本1.3开始，可以将CachingConnectionFactory配置为缓存连接以及通道。在这种情况下，每次调用createConnection（）都会创建一个新连接（或从缓存中检索一个空闲连接）关闭连接会将其返回到缓存（如果尚未达到缓存大小）。在这种连接上创建的通道也被缓存

并不限制连接数量，它指定允许多少空闲打开的连接。

从版本1.5.5开始，提供了一个新的属性connectionLimit。设置此项时，会限制允许的连接总数,设置后，如果达到限制，则使用channelCheckoutTimeLimit等待连接变为空闲状态。如果超过该时间，则抛出AmqpTimeoutException。

当缓存模式为CONNECTION时，不支持自动声明队列等（请参见“自动声明交换，队列和绑定”一节）。此外，在撰写本文时，rabbitmq-client库默认为每个连接（5个线程）创建一个固定的线程池。在使用大量连接时，应考虑在CachingConnectionFactory上设置自定义执行程序,然后，所有连接将使用相同的执行程序，并且它的线程可以共享。执行程序的线程池应该是无界的，或者为预期的利用率设置适当的值（通常每个连接至少有一个线程）,如果在每个连接上创建多个通道，则池大小将影响并发性，因此可变的（或简单缓存）的线程池执行程序将是最合适的。

重要的是理解cache size不是限制，而是可以缓存通道的数量， 客户可以使用更多的频道，但是超过这个数字的话他们不会被缓存，比如设置为10，如果信道超过10，10个进入缓存，其他的不会被缓存

从版本1.6开始，默认通道高速缓存大小从1增加到25.在大容量，多线程环境中，小型高速缓存意味着以高速率创建和关闭通道。增加默认缓存大小可以避免这种开销。您应该通过RabbitMQ Admin用户界面监控正在使用的频道，并且如果看到许多频道正在创建和关闭，请考虑进一步增加缓存大小。缓存只会按需增长（以适应应用程序的并发需求），因此这种更改不会影响现有的低容量应用程序。

从版本1.4.2开始，CachingConnectionFactory具有一个属性channelCheckoutTimeout。当此属性大于零时，channelCacheSize将成为可在连接上创建的通道数的限制。如果达到限制，调用线程将会阻塞，直到有一个通道可用或达到此超时为止，在这种情况下会引发AmqpTimeoutException。

框架内使用的通道（例如RabbitTemplate）将可靠地返回到缓存。如果您在框架之外创建通道（例如，直接访问连接并调用createChannel（）），您必须可靠地返回它们（通过关闭），也许在最后一个块中，以避免用尽频道。

```java
CachingConnectionFactory connectionFactory = new CachingConnectionFactory("somehost");
connectionFactory.setUsername("guest");
connectionFactory.setPassword("guest");
Connection connection = connectionFactory.createConnection();
```
```xml
<bean id="connectionFactory"
      class="org.springframework.amqp.rabbit.connection.CachingConnectionFactory">
    <constructor-arg value="somehost"/>
    <property name="username" value="guest"/>
    <property name="password" value="guest"/>
</bean>
```

还有一个SingleConnectionFactory实现，它只在框架的单元测试代码中可用，它比CachingConnectionFactory简单，因为它不会缓存频道，但由于缺乏性能和弹性，它不适用于简单测试之外的实际应用。如果由于某种原因需要实现自己的ConnectionFactory，那么AbstractConnectionFactory基类可能会提供一个很好的起点。

通过[rabbitMq命名空间][rabbitMq命名空间]，可以查看添加属性

这里有一个自定义线程工厂的例子，它使用rabbitmq-作为线程名称的前缀。
```xml
<rabbit:connection-factory id="multiHost" virtual-host="/bar" addresses="host1:1234,host2,host3:4567"
    thread-factory="tf"
    channel-cache-size="10" username="user" password="password" />

<bean id="tf" class="org.springframework.scheduling.concurrent.CustomizableThreadFactory">
    <constructor-arg value="rabbitmq-" />
</bean>
```

从版本1.7开始，为注入到AbstractionConnectionFactory中提供了一个ConnectionNameStrategy。生成的名称用于目标RabbitMQ连接的应用程序特定标识。如果RabbitMQ服务器支持，连接名称将显示在管理UI中。该值不必是唯一的，并且不能用作连接标识符,例如在HTTP API请求中。此值应该是人类可读的，并且是connection_name键下的ClientProperties的一部分

ConnectionFactory参数可用于通过某种逻辑区分目标连接名称。默认情况下，使用AbstractConnectionFactory的beanName和内部计数器来生成connection_name,<rabbit：connection-factory>命名空间组件也随连接名称策略属性提供。

从版本1.7.7开始，提供了当SimpleConnection.createChannel（）不能创建一个Channel的AmqpResourceNotAvailableException，因为达到了通道最大限制，并且缓存中没有可用通道。这个异常可以在RetryPolicy中使用，以在一些退避后恢复操作。

## Configuring the Underlying Client Connection Factory 配置基础客户端连接工厂

### 

# 参考
[SpringAMPQ官方文档][SpringAMPQ官方文档]

[SpringAMPQ官方API][SpringAMPQ官方API]

[SpringAMPQ官方文档]:https://docs.spring.io/spring-amqp/docs/1.7.8.RELEASE/reference/html/_reference.html

[SpringAMPQ官方API]:https://docs.spring.io/spring-amqp/docs/1.7.8.RELEASE/api/

[rabbitMq命名空间]:http://www.springframework.org/schema/rabbit/spring-rabbit-1.2.xsd