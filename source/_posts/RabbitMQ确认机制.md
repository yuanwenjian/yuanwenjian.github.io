---
title: RabbitMQ消息确认
description: RabbitMQ是一个消息代理 - 一个消息系统的媒介。它可以为你的应用提供一个通用的消息发送和接收平台，并且保证消息在传输过程中的安全。
date: 2018-05-08 12:30:00
comments: true
tags: 
    - RabbitMQ
    - 后端  
categories:
    - 后端
---
# 介绍
从消费者到RabbitMQ的递送处理确认被称为消费确认;borker对生产者的确认称为生产者确认。这两个功能都基于相同的想法，并受TCP的启发。它们对于从发行商到RabbitMQ节点以及从RabbitMQ节点到消费者的可靠传送都是至关重要的



ReturnCallback 确认到队列
# 消费端确认

## Delivery Tags

在我们继续讨论其他主题之前，重要的是解释如何识别交付（并且确认指示它们各自的交付），当消费者或者订阅者被注册时，消息将由RabbitMQ的basic.deliver方法传递（推送）,该方法会携带一个channel唯一标识的deliberyTag，
DiliveryTags 是单方面增长的正整数，并由客户端库提供。确认交付的客户端库方法将交付标签作为参数。
由于DiliveryTag的范围为Channel,所以必须在同一channel确定，在不同的信道确认会导致unknown delivery tag错误

## 消费端确认模式与数据安全注意事项

当节点向消费者传递消息时，它必须决定消息是否应该被消费者视为处理（或至少是接收到）。由于多种情况（客户端连接，消费者应用程序等）可能会失败，这种是一种注意数据安全的方案，消息传递协议通常提供一种确认机制，允许消费者确认交付给他们所连接的节点。该机制是否被使用是在消费者订阅时决定的。

根据所使用的确认模式，RabbitMQ可以考虑将消息发送出去（写入TCP套接字）后立即或者在收到显式（“手动”）客户端确认后立即成功发送，数据确认可以是正向或反向，及确认接收或拒绝接收，通过以下协议完成

**basic.ack**用于肯定确认 

**basic.nack**用于否定确认（注意：这是对AMQP 0-9-1的RabbitMQ扩展） 

**basic.reject**用于否定确认，但与basic.nack相比有一个限制

下面将讨论如何在客户端库API中使用这些方法。

积极的确认只是指示RabbitMQ记录已发送的消息并且可以放弃。使用basic.reject的否定确认具有相同的效果，差异主要在语义上：积极的承认假设一条消息被成功处理，而消极的对方认为交付没有被处理，但仍然应该被删除。

在自动确认模式下，消息被认为在发送后立即成功发送。他的模式换来更高的吞吐量（只要消费者能够保持），以降低交付和消费者处理的安全性。这种模式通常被称为“即忘即忘”。与手动确认模型不同，如果消费者的TCP连接或频道在成功发送前关闭，则服务器发送的消息将会丢失。因此，自动消息确认应被<font color='red'>**视为不安全**</font>并且不适用于所有工作负载。

使用自动确认模式时需要考虑的另一件事是消费者过载,手动确认模式通常限制通道预取，限制通道上未完成（“正在进行中”）传送的次数,然而，自动确认没有这样的限制。因此，消费者可能会因交付速度而感到不知所措，可能积累了内存中的积压，堆积如山，或者由操作系统终止了进程，某些客户端库将应用TCP反压（在未处理的交付积压下降超过一定限制之前停止从套接字读取数据）。因此，仅建议可以高效且稳定处理交货的消费者使用自动交付方式。

## 积极确认

用于传递确认的API方法通常作为客户端库中的通道的操作公开。Java客户端用户将分别使用Channel＃basicAck和Channel＃basicNack来执行basic.ack和basic.nack。以下是一个Java客户端示例，展示了一个积极的承认
```java
boolean autoAck = false;
channel.basicConsume(queueName, autoAck, "a-consumer-tag",
     new DefaultConsumer(channel) {
         @Override
         public void handleDelivery(String consumerTag,
                                    Envelope envelope,
                                    AMQP.BasicProperties properties,
                                    byte[] body)
             throws IOException
         {
             long deliveryTag = envelope.getDeliveryTag();
             // positively acknowledge a single delivery, the message will
             // be discarded
             channel.basicAck(deliveryTag, false);
         }
     });
```

## 一次确认多次

可以批量手动确认以减少网络流量。这是通过将确认方法的多个字段（见上文）设置为true来完成的。注意，basic.reject在历史上并没有这个字段，这就是为什么basic.nack被RabbitMQ作为协议扩展引入的原因。

当多字段设置为true时，RabbitMQ将确认这个序列号之前的所有消息都已经得到了处理。，直至包括确认中指定的标签。与其他所有与确认相关的内容一样，这是每个channel的范围,比如ch信道中有5,6,7,8tag,当确认8时，mutiple设为true,该信道里所有tag状态全部变为acknowledged，如果设为false,5，6,7状态未unacknowledged

## 消极确认和重新入队列
有时消费者不能立即处理交付，但其他实例可能能够处理交付。在这种情况下，可能需要重新安排并让其他消费者接收并处理它。 basic.reject和basic.nack是两种协议方法。

这些方法通常用于否定确认交付。这种交付可以被broker丢弃或重新进入队列,此行为由requeue字段控制。当该字段设置为true时，代理将使用指定的交付标签重新进行交付（或多次交付，稍后将作解释）。

这两种方法通常在客户端库中作为频道的操作公开。 Java客户端用户将分别使用Channel＃basicReject和Channel＃basicNack来执行basic.reject和basic.nack：

```java
boolean autoAck = false;
channel.basicConsume(queueName, autoAck, "a-consumer-tag",
     new DefaultConsumer(channel) {
         @Override
         public void handleDelivery(String consumerTag,
                                    Envelope envelope,
                                    AMQP.BasicProperties properties,
                                    byte[] body)
             throws IOException
         {
             long deliveryTag = envelope.getDeliveryTag();
             // negatively acknowledge, the message will be discarded
             channel.basicReject(deliveryTag, false);
             // requeue the delivery
             channel.basicReject(deliveryTag, true)
         }
     });
```

如果可能，当消息被重新发送时，它将被放置在其队列中的原始位置,否则（由于多个消费者共享队列时来自其他消费者的并发递送和确认），则该消息将被重新排队到更接近队列头的位置

已重新发送的消息可能会立即准备好重新发送，具体取决于它们在队列中的位置，以及具有活动使用者的通道使用的预取值。这意味着，如果所有消费者因为暂时状况而无法处理交货而需要处理，他们将创建一个重新发货/递送循环。就网络带宽和CPU资源而言，这样的循环可能是昂贵的。消费者实现可以跟踪重新传送的次数并拒绝好的消息（丢弃它们）或者延迟后计划重新计划


使用basic.nack方法可以一次性拒绝或重新发送多个消息。这是区别于basic.reject的。它接受一个额外的参数，multiple。以下是一个Java客户端示例：
```java
boolean autoAck = false;
channel.basicConsume(queueName, autoAck, "a-consumer-tag",
     new DefaultConsumer(channel) {
         @Override
         public void handleDelivery(String consumerTag,
                                    Envelope envelope,
                                    AMQP.BasicProperties properties,
                                    byte[] body)
             throws IOException
         {
             long deliveryTag = envelope.getDeliveryTag();
             // requeue all unacknowledged deliveries up to
             // this delivery tag
             channel.basicNack(deliveryTag, true, true);
         }
     });
```

## channel设置预读取限制

由于消息是异步发送（推送）给客户端的，因此在任何给定时刻通常都有不止一条消息在信道上，此外，客户的手动确认本质上也是异步的。所以有一个未确认的交付标签的限制大小。开发人员通常会倾向于限制此窗口的大小，以避免消费者端无限制的缓冲区问题。这是通过使用basic.qos方法设置“预取计数”值完成的。该值定义了通道上允许的最大未确认递送数量,一旦数字达到配置的计数，RabbitMQ将停止在通道上传送更多消息，除非至少有一个未确认的消息被确认。

例如，假设在通道Ch上未确认的递送标签5,6,7和8以及通道Ch的预取计数设置为4，除非至少有一个未完成交付得到确认，否则RabbitMQ将不会再推送更多交付。当确认帧在delivery_tag设置为8的频道上到达时，RabbitMQ将会注意到并再发送一条消息。

值得重申的是，交付流程和手动客户确认完全是异步的。因此，如果在飞行中已经有交付事件的情况下预取值发生变化，则会出现自然的竞争状况，并且暂时可能比预取计数通道上未确认的消息更多。

QoS预取设置对使用basic.get（“pull API”）获取的消息没有影响，即使在手动确认模式下也是如此。

QoS设置可以为频道或消费者配置。有关详细信息，请参阅[消费预取][消费预取设置]。

## 消费者确认模式，预取和吞吐量

确认模式和QoS预取值对消费者吞吐量有显着影响

## 消费者确认模式，预取和吞吐量

确认模式和QoS预取值对消费者吞吐量有显着影响。一般来说，增加预取将提高向消费者传递消息的速度.自动确认模式可以产生最佳的传送速率。但是，在这两种情况下，交付但尚未处理的消息数量也会增加，从而增加消费者RAM消耗。

自动确认模式或带无限预取的手动确认模式应谨慎使用。消费者在没有确认的情况下消耗大量消息将导致其所连接的节点上的内存消耗增长。寻找合适的预取值是一个试验和错误的问题，并且会因工作负载而异。 100到300范围内的值通常提供最佳的吞吐量，并且不会面临压倒性消费者的重大风险.更高的价值往往会遇到[收益递减的规律][收益递减的规律]。

1的预取值是最保守的。这将显着降低吞吐量，特别是在消费者连接延迟较高的环境中。对于许多应用来说，更高的价值是合适和最佳的。

## 消费者失败或失去连接时：自动重新进入队列

使用手动确认时，任何未收到的递送（消息）将在发送递送的通道（或连接）关闭时自动重新发送。这包括客户端的TCP连接丢失，消费者应用程序（进程）失败以及通道级别的协议例外（如下所述）。

注意，[检测不可用客户端][检测不可用客户端]需要一段时间。

由于这种行为，消费者必须做好准备来处理重新购买决策，否则就必须考虑[幂等][幂等]
Redeliveries将有一个特殊的布尔属性，重新传递，由RabbitMQ设置为true。首次发货时，它将被设置为false。请注意，消费者可以收到先前传送给其他消费者的消息。

## 客户端错误：双重确认和未知标签

如果客户不止一次确认同一个递送标签，RabbitMQ将导致通道错误,如PRECONDITION_FAILED - unknown delivery tag 100. 如果使用未知的交付标签，则会抛出相同的异常。

broker报unknown delivery tag异常的另一种情况为尝试在不同于信道中确认。确认必须在同一信道

# 生产端确认

网络可能以不甚明显的方式失败，并且检测到某些故障需要时间。因此，向其套接字写入协议帧或一组帧（例如发布的消息）的客户端不能认为该消息已到达服务器并且已成功处理。它可能一直在丢失，或者它的消费可能会显着延迟。

使用标准的AMQP 0-9-1，保证消息不会丢失的唯一方法是使用事务 - 使通道事务化，然后为每个消息或消息集发布，提交。在这种情况下，交易不必要地增加重量并将吞吐量降低250倍。为了解决这个问题，引入了确认机制。它模仿协议中已经存在的消费者确认机制。

要启用确认，客户端会发送confirm.select方法。根据是否设置不等待，代理可以用confirm.select-ok进行响应.一旦在通道上使用confirm.select方法，就说它处于确认模式。事务通道不能进入确认模式，一旦通道处于确认模式，就不能进行事务处理。两者只能使用一种

一旦通道处于确认模式，代理和客户端都会对消息进行计数（在第一个confirm.select中从1开始计数）.然后，broker通过在相同频道上发送basic.ack来处理消息，从而确认消息。投放标签字段包含确认消息的序列号。代理也可以在basic.ack中设置多个字段，以指示所有消息直到并包括具有序列号的消息都已被处理。

## 反向确认
在特殊情况下，broker无法成功处理消息，而不是basic.ack，经纪人将发送一个basic.nack。在这种情况下，basic.nack的字段与basic.ack中的相应字段具有相同的含义，并且应忽略requeue字段。通过隐藏一条或多条消息，broker表示无法处理消息并拒绝对他们负责;那时，客户可以选择重新发布消息。

通道进入确认模式后，所有随后发布的消息都将被确认或删除一次.无法保证消息的确认时间。没有一条信息既是confirmed又是 nack'd.

只有在负责队列的Erlang进程中发生内部错误时才会传递basic.nack。

## broker将何时确定消息？

对于不可路由的消息，一旦交换验证消息不会路由到任何队列（返回空的队列列表），代理将发出确认。如果该消息也是强制发布的，basic.return会在basic.ack之前发送给客户端。对于否定确认（basic.nack）也是如此。

对于可路由消息，当消息被所有队列接受时发送basic.ack。对于路由到持久队列的持久消息，这意味着持久化到磁盘.对于镜像队列，这意味着所有镜像都接受了该消息。

## 持久化信息延迟

将消息保存到磁盘后，将发送路由到持久队列的持久消息的basic.ack。RabbitMQ消息存储在间隔（几百毫秒）后将消息持久保存到磁盘，以最大限度地减少fsync（2）调用的次数，或者当一个队列处于空闲状态时。这意味着在恒定负载下，basic.ack的延迟可能会达到几百毫秒。为了提高吞吐量，强烈建议应用程序异步处理确认（作为流）或发布批量消息并等待未完成的确认。客户端库的确切API不尽相同。

## 生产者确认顺序

在大多数情况下，RabbitMQ将按发布顺序向生产者确认消息（这适用于在单个频道上发布的消息）。但是，发布者确认是异步发出的，并且可以确认一条消息或一组消息。发出确认的确切时刻取决于消息的传递模式（持久性与瞬态）以及消息被路由到的队列的属性（请参见上文）。也就是说不同的消息可以被认为准备好在不同的时间进行确认。这意味着，与各自的消息相比，确认可以以不同的顺序到达。应用程序应尽可能不取决于确认的顺序。

## 确认并保证投放

如果代理在所述消息写入磁盘之前崩溃，代理将丢失持久性消息。在某些情况下，这会导致broker以令人惊讶的方式行事。

考虑这种情况：

1. 客户端发布持久消息给持久队列
2. 客户端使用队列中的消息（注意消息是持久的并且队列持久），但还没有确认，
3. broker死亡并重新启动
4. 客户端重新连接并开始消费消息。

此时，客户可以合理地认为该消息将再次发送。但是重新启动导致代理丢失消息。为了保证持久性，客户应该使用确认,如果发布者的频道处于确认模式，则发布者不会收到丢失消息的确认（因为消息尚未写入磁盘）。

# 限制

## Delivery Tag
递送标签是一个64位长的值，因此其最大值为9223372036854775807.由于Delivery Tag的范围是按每个通道划分的，因此发布商或消费者在实践中不太可能运行该值。

# 参考
[RabbitMQ确认机制][确认机制]

[确认机制]:https://www.rabbitmq.com/confirms.html
[消费预取设置]:https://www.rabbitmq.com/consumer-prefetch.html
[收益递减的规律]:https://www.rabbitmq.com/blog/2014/04/14/finding-bottlenecks-with-rabbitmq-3-3/
[检测不可用客户端]:https://www.rabbitmq.com/heartbeats.html

[幂等]:https://baike.baidu.com/item/%E5%B9%82%E7%AD%89/8600688?fr=aladdin
## 带确认
basic.nack被RabbitMQ作为协议扩展引入的原因。

生产者确定

镜像队列