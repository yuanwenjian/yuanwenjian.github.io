---
title: RabbitMQ笔记
description: RabbitMQ是一个消息代理 - 一个消息系统的媒介。它可以为你的应用提供一个通用的消息发送和接收平台，并且保证消息在传输过程中的安全。
date: 2018-06-10 12:30:00
comments: true
tags: 
    - RabbitMQ
    - 后端  
categories:
    - 后端
---

# RabbitMq简介

# 简介
消息系统允许软件、应用相互连接和扩展．这些应用可以相互链接起来组成一个更大的应用，或者将用户设备和数据进行连接．消息系统通过将消息的发送和接收分离来实现应用程序的异步和解偶．

或许你正在考虑进行数据投递，非阻塞操作或推送通知。或许你想要实现发布／订阅，异步处理，或者工作队列。所有这些都可以通过消息系统实现。

RabbitMQ是一个消息代理 - 一个消息系统的媒介。它可以为你的应用提供一个通用的消息发送和接收平台，并且保证消息在传输过程中的安全。

## 技术亮点
1. 可靠性
RabbitMQ提供了多种技术可以让你在性能和可靠性之间进行权衡。这些技术包括持久性机制、投递确认、发布者证实和高可用性机制。

2. 灵活的路由
消息在到达队列前是通过交换机进行路由的。RabbitMQ为典型的路由逻辑提供了多种内置交换机类型。如果你有更复杂的路由需求，可以将这些交换机组合起来使用，你甚至可以实现自己的交换机类型，并且当做RabbitMQ的插件来使用。

3. 集群
在相同局域网中的多个RabbitMQ服务器可以聚合在一起，作为一个独立的逻辑代理来使用。

4. 联合
对于服务器来说，它比集群需要更多的松散和非可靠链接。为此RabbitMQ提供了联合模型。

5. 高可用的队列
在同一个集群里，队列可以被镜像到多个机器中，以确保当其中某些硬件出现故障后，你的消息仍然安全。

6. 多协议
RabbitMQ 支持多种消息协议的消息传递。

7. 广泛的客户端
只要是你能想到的编程语言几乎都有与其相适配的RabbitMQ客户端。

8. 可视化管理工具
RabbitMQ附带了一个易于使用的可视化管理工具，它可以帮助你监控消息代理的每一个环节。

9. 追踪
如果你的消息系统有异常行为，RabbitMQ还提供了追踪的支持，让你能够发现问题所在。

10. 插件系统
RabbitMQ附带了各种各样的插件来对自己进行扩展。你甚至也可以写自己的插件来使用。

# RabbitMQ架构图

![RabbitMQ][rabbitMq]

# RabbitMQ概念介绍
1. Producing , 生产者, 产生消息的角色. 对应上图P
2. Exchange , 交换机,用于转发消息，但是它不会做存储_ ，如果没有 Queue bind 到 Exchange 的话，它会直接丢弃掉 Producer 发送过来的消息。
    - **路由键**: 消息到交换机的时候，交互机会转发到对应的队列中，那么究竟转发到哪个队列，就要根据该路由键。

3. Queue , 队列, 消息暂时呆的地方. 对应上图红色队列
4. Consuming , 消费者, 把消息从队列中取出的角色. 对应上图C
5. Message , RabbitMQ 中的消息有自己的一系列属性, 某些属性对信息流有直接影响.
6.  绑定：也就是交换机需要和队列相绑定，这其中如上图所示，是多对多的关系。

## 交换机详解
rabbitmq的message model实际上消息不直接发送到queue中，中间有一个exchange是做消息分发，producer甚至不知道消息发送到那个队列中去。因此，当exchange收到message时，必须准确知道该如何分发。是append到一定规则的queue，还是append到多个queue中，还是被丢弃？这些规则都是通过exchagne的4种type去定义的。

### Directed Exchange

直连型交换机（direct exchange）是根据消息携带的路由键（routing key）将消息投递给对应队列的。直连交换机用来处理消息的单播路由（unicast routing）（尽管它也可以处理多播路由）。下边介绍它是如何工作的：

将一个队列绑定到某个交换机上，同时赋予该绑定一个路由键（routing key）
当一个携带着路由键为R的消息被发送给直连交换机时，交换机会把它路由给绑定值同样为R的队列。
直连交换机经常用来循环分发任务给多个工作者（workers）。当这样做的时候，我们需要明白一点，在AMQP 0-9-1中，消息的负载均衡是发生在消费者（consumer）之间的，而不是队列（queue）之间。
路由键exchange，该交换机收到消息后会把消息发送到指定routing-key的queue中。那消息交换机是怎么知道的呢？其实，producer deliver消息的时候会把routing-key add到 message header中。routing-key只是一个messgae的attribute。

### Topic Exchange
主题交换机（topic exchanges）通过对消息的路由键和队列到交换机的绑定模式之间的匹配，将消息路由给一个或多个队列。主题交换机经常用来实现各种分发/订阅模式及其变种。主题交换机通常用来实现消息的多播路由（multicast routing）。主题交换机拥有非常广泛的用户案例。无论何时，当一个问题涉及到那些想要有针对性的选择需要接收消息的 多消费者/多应用（multiple consumers/applications） 的时候，主题交换机都可以被列入考虑范围。

使用案例：
- 分发有关于特定地理位置的数据，例如销售点
- 由多个工作者（workers）完成的后台任务，每个工作者负责处理某些特定的任务
- 股票价格更新（以及其他类型的金融数据更新）
- 涉及到分类或者标签的新闻更新（例如，针对特定的运动项目或者队伍）
- 云端的不同种类服务的协调
- 分布式架构/基于系统的软件封装，其中每个构建者仅能处理一个特定的架构或者系统。
- 匹配多个word和路径，路径之间通过.隔开。如满足a._.c的routing-key有a.hello.c；满足#.hello的routing-key有a.b.c.helo。

### Fanout Exchange

扇型交换机（funout exchange）将消息路由给绑定到它身上的所有队列，而不理会绑定的路由键。如果N个队列绑定到某个扇型交换机上，当有消息发送给此扇型交换机时，交换机会将消息的拷贝分别发送给这所有的N个队列。扇型用来交换机处理消息的广播路由（broadcast routing）。

因为扇型交换机投递消息的拷贝到所有绑定到它的队列，所以他的应用案例都极其相似：

- 大规模多用户在线（MMO）游戏可以使用它来处理排行榜更新等全局事件
- 体育新闻网站可以用它来近乎实时地将比分更新分发给移动客户端
- 分发系统使用它来广播各种状态和配置更新
- 在群聊的时候，它被用来分发消息给参与群聊的用户。（AMQP没有内置presence的概念，因此XMPP可能会是个更好的选择）

### Header Exchange
设置header attribute参数类型的交换机

## 队列
AMQP中的队列（queue）跟其他消息队列或任务队列中的队列是很相似的：它们存储着即将被应用消费掉的消息。队列跟交换机共享某些属性，队列在声明（declare）后才能被使用。如果一个队列尚不存在，声明一个队列会创建它。如果声明的队列已经存在，并且属性完全相同，那么此次声明不会对原有队列产生任何影响。如果声明中的属性与已存在队列的属性有差异，那么一个错误代码为406的通道级异常就会被抛出。

## 绑定
绑定（Binding）是交换机（exchange）将消息（message）路由给队列（queue）所需遵循的规则。如果AMQP的消息无法路由到队列（例如，发送到的交换机没有绑定队列），消息会被就地销毁或者返还给发布者。如何处理取决于发布者设置的消息属性。

## 虚拟主机
为了在一个单独的代理上实现多个隔离的环境（用户、用户组、交换机、队列 等），AMQP提供了一个虚拟主机（virtual hosts - vhosts）的概念。这跟Web servers虚拟主机概念非常相似，这为AMQP实体提供了完全隔离的环境。当连接被建立的时候，AMQP客户端来指定使用哪个虚拟主机。

## 通道
有些应用需要与AMQP代理建立多个连接。无论怎样，同时开启多个TCP连接都是不合适的，因为这样做会消耗掉过多的系统资源并且使得防火墙的配置更加困难。AMQP 0-9-1提供了通道（channels）来处理多连接，可以把通道理解成共享一个TCP连接的多个轻量化连接。

# RabbitMQ持久化
## 持久化前提
要从奔溃的 RabbitMQ 中恢复的消息，我们需要做消息持久化。如果消息要从 RabbitMQ 奔溃中恢复，那么必须满足三点，且三者缺一不可。

1. 交换器必须是持久化。
2. 队列必须是持久化的。
3. 消息必须是持久化的。

# 消息投递流程

## 流程图
![消息投递流程图][消息投递流程图]

## 
## SpringRabbitMQ持久化代码

使用SpringRabbitMQ不需要指定，默认以上三个均为持久化
代码如下：
```xml
交换机
<xsd:complexType name="exchangeType">
    ...
    <xsd:attribute name="durable" use="optional" type="xsd:string " default="true">
        <xsd:annotation>
            <xsd:documentation><![CDATA[
                Flag indicating that the exchange is durable, i.e. will survivbroker restart.  Default is true.]]>
            </xsd:documentation>
        </xsd:annotation>
    </xsd:attribute>
    ...
</xsd:complexType>
队列
<xsd:element name="queue">
......
    <xsd:attribute name="durable" use="optional" type="xsd:string" default="true">
        <xsd:annotation>
            <xsd:documentation><![CDATA[
            Flag indicating that the queue is durable, meaning that it wisurvive broker restarts (not that the messages in it wilalthough they might if they are persistent).  Default is true.
        ]]></xsd:documentation>
        </xsd:annotation>
    </xsd:attribute>
......
</xsd:element>
```
```java
消息模式
org.springframework.amqp.rabbit.core.RabbitTemplate
public void convertAndSend(String exchange, String routingKey, Object object, CorrelationData correlationData) throws AmqpException {
        this.send(exchange, routingKey, this.convertMessageIfNecessary(object), correlationData);
}

protected Message convertMessageIfNecessary(Object object) {
        return object instanceof Message ? (Message)object : this.getRequiredMessageConverter().toMessage(object, new MessageProperties());
}

org.springframework.amqp.core.MessageProperties
public MessageProperties() {
        this.deliveryMode = DEFAULT_DELIVERY_MODE;
        this.priority = DEFAULT_PRIORITY;
}
static {
        DEFAULT_DELIVERY_MODE = MessageDeliveryMode.PERSISTENT;
        DEFAULT_PRIORITY = 0;
}
```
# SpringBoot与RabbitMQ整合Demo

[SpringBoot整合MQ Demo][github]
# 参考
[RabbitMQ中文文档][rabbit官网]

[Spring 集成 RabbitMQ 与其概念，消息持久化，ACK机制等][Spring整合RabbitMQ]

[rabbitMq]:/images/rabbitMQ.png
[消息投递流程图]:/images/投递消息.png

[rabbit官网]: http://rabbitmq.mr-ping.com/description.html
[Spring整合RabbitMQ]:https://github.com/401Studio/WeekLearn/issues/2
[github]: https://github.com/yuanwenjian/spring-mq