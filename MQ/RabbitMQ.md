<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [RabbitMQ 知识点](#rabbitmq-知识点)
  - [RabbitMQ 的一些知识点](#rabbitmq-的一些知识点)
    - [消费者预取](#消费者预取)
    - [RabbitMQ 基于 AMQP 消息协议](#rabbitmq-基于-amqp-消息协议)
    - [死信](#死信)
    - [延时队列](#延时队列)
      - [RabbitMQ 中的 TTL](#rabbitmq-中的-ttl)
    - [消息重复消费](#消息重复消费)
    - [RabbitMQ 的五种消息类型](#rabbitmq-的五种消息类型)
      - [Hello World](#hello-world)
      - [Work queues](#work-queues)
      - [Publish/Subscribe(发布订阅)](#publishsubscribe发布订阅)
      - [Routing](#routing)
      - [Topics](#topics)
  - [RabbitMQ 高频的面试题](#rabbitmq-高频的面试题)

<!-- /code_chunk_output -->

# RabbitMQ 知识点

Date：2024-11-05

author：Cary

RabbitMQ 是消息中间件，常用于异步、削峰、解耦。

RabbitMQ 中有一些术语：

- 生产者（producer）： 生产者意味着只发送消息的程序

- 队列（consumer）：

许多生产者可以发送到一个队列消息，许多消费者也可以从一个队列消费

> queue_name 表示队列的名称

- 消费者： 一个程序只等待接收消息

## RabbitMQ 的一些知识点

### 消费者预取

Consumer Prefetch 是消费者在处理消息之前可以预先从队列中获取的消息数量。

RabbitMQ 会根据消费者设置的预取数量来发送消息，消费者处理完一条消息会发送 ack，MQ 会再次发送新的消息给消费者。

### RabbitMQ 基于 AMQP 消息协议

### 死信

死信队列就是无法被消费的队列，一般来说，生产者将消息投递到 queue 中，消费者会从 queue 消费消息，但由于特定原因导致 queue 中的消息无法被消费，这样的消费没有后续处理就变成了死信。

### 延时队列

延时队列常用于在指定时间到了以后或以前需要取出和处理的消息。

例如使用场景为：

- 订单十分钟之内未支付则自动取消

- 预定会议后，在某个时间点通知参会人，

#### RabbitMQ 中的 TTL

TTL 是一个队列或消息的属性，表明一条消息，或一个队列中所有消息的最大存活时间，如果超出设置的时间没有消费，该消息会成为死信。

利用 TTL 的特性，延迟队列就可以实现。

### 消息重复消费

在消息队列中，如果一个队列把消息发送给消费者，消费者在返回 ack 时网络中断没返回，mq 未收到消息会转发给其他的消费者，造成消息的重复消费。

如何解决呢？

通过唯一标识符，判断消息的唯一标识符来确认是否处理过。

### RabbitMQ 的五种消息类型

#### Hello World

#### Work queues

1，在工作队列中，会有多个消费者消费一个队列的消息，队列的规则是循环调度，会向每个消费者轮流推送消息。

可以通过预设数量来限制消费者每次消费的消息数量。

2，**消息确认机制（ack）**

工作队列有消息确认机制，为了防止消费者死亡导致消息丢失的情况。一条消息到达消费者会告诉 RabbitMQ 收到消息并且可以让 RabbitMQ 自由删除消息了。

如果 RabbitMQ 没有收到消费者的确认，将会把消息重新排队，交给另外的消费者

默认的确认时间超时为 30 分钟，也可以通过 ` Delivery Acknowledgement Timeout` 增加超时时间。

3，**消息持久化机制**

使用消息确认机制可以保证消费者死亡时，消息不会丢失，但无法保证在消息到达 RabbitMQ 后，RabbitMQ 死亡导致消息丢失。

使用 `channel.queueDeclare("you_queuename",true,false, false, null)` 这个方法的第二个参数是开启队列持久化

> 如果你曾经创建了一个名为 test 的 queue，但是并没开启持久化，就修改了代码开启 test 的 queue 持久化，并不会直接发生改变而是会报错，你需要删除原来的 queue 或者新建 queue。

消息持久化需要设置

```
channel.basicPublish("", "task_queue",
            MessageProperties.PERSISTENT_TEXT_PLAIN,
            message.getBytes());
```

> **关于消息持久性的注意事项**
> 将消息标记为持久性并不能完全保证消息 不会丢失。虽然它告诉 RabbitMQ 将消息保存到磁盘， RabbitMQ 接受消息的时间窗口仍然很短，并且 尚未保存。此外，RabbitMQ 并不适用于每个 message —— 它可能只是保存到缓存中，而不是真正写入 磁盘。持久性保证并不强大，但已经绰绰有余 对于我们的简单任务队列。如果您需要更强的保证，那么您可以使用 publisher confirms。

4，公平调度

在 Work queue 情况下，如果奇数的消息很大，但是偶数的消息很小，会导致处理速度有偏差，但是 RabbitMQ 还是会按照规则一个一个消息的轮流发送给消费者。这会导致一个消费者很忙，一个消费者很闲，为了防止这种情况，RabbitMQ 可以设置

```
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```

这段代码会将消费者进行限流，在消费者没处理完数据向 RabbitMQ 发送 ask 时，RabbitMQ 不会再给这个消费者推送消息。

#### Publish/Subscribe(发布订阅)

在工作队列中，我们始终一条消息只有一个消费者能接受到。发布订阅模式不同的是，一条消息会有多个消费者能收到。

1，**交换器（Exchange）**

生产者会将消息发送给 Exchange，Exchange 会接收他们的消息，并将消息推送给队列。

![](https://upload-images.jianshu.io/upload_images/27448672-a715da3f462dac92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

交换机的类型一共有四种：分别是：direct、topic、headers、fanout。本章主要讲解最后一种：fanous（广播模式）。

解读：

1、1 个生产者，多个消费者

2、每一个消费者都有自己的一个队列

3、生产者没有将消息直接发送到队列，而是发送到了交换机

4、每个队列都要绑定到交换机

5、生产者发送的消息，经过交换机，到达队列，实现，一个消息被多个消费者获取的目的

声明交换器

```
channel.exchangeDeclare("logs", "fanout");
```

发送消息

```
channel.basicPublish( "logs", "", null, message.getBytes());
```

在工作队列中，队列命名是特别重要的，因为消费者需要根据命名来消费队列中的消息，但是在交换机中，每个消费者都会创建一个临时队列，命名是随机的，并且当消费者离线时，队列会自动删除。

```
String queueName = channel.queueDeclare().getQueue();
```

> 需要注意的是，交换机没有储存消息的能力，如果消息发送给了没有队列绑定的交换机，消息会丢失。

#### Routing

路由模式基于交换机模式做了一些拓展，在生产者发送消息到交换机时，会带上一个路由键，交换机根据路由键来决定将消息路由到哪个队列。

#### Topics

主题模式则在交换机向队列转发消息时做了更灵活的限制，可以使用通配符等，制定转发规则。

## RabbitMQ 高频的面试题

- 消息有几种应答模式？

两种，一种是默认的自动应答，一种是手动应答。

- 你了解 RabbitMQ 的持久化吗？

了解，在使用 RabbitMQ 时，队列或消息会因为 rabbitmq 的关闭消失，在创建队列时可以使用 durable 的参数设置为 true，创建持久化队列。

而消息则需要在发送消息时设置持久化消息

- 你知道怎么解决不公平分发吗？

知道，通过设置 basicQos（），通过限流方式，让消费者消息处理完了以后再发送 ack 接收新的消息。

- 死信是什么，什么原因造成的，怎么解决？

无法被消费的消息，queue 里的消息无法被消费者消费，没有后续处理就会变成死信，会有死信队列来存储死信。

造成死信的原因主要有三个：

1，消息 TTL 过期

2，超过最大队列长度

3，消息被拒绝

可以建立一个消费者专门消费死信队列，进行错误处理，或者重发等方式。

- RabbitMQ 延迟队列是靠什么实现的？

是靠队列的 TTL 属性，消息过期后会被扔到死信队列中，消费者只需要消费死信队列中的消息即可。

- 如何保证消息消费的幂等性？

使用唯一标识符，消息到达消费者时，根据唯一标识符判断 redis 里是否处理过该消息，如果没处理过，就将消息处理后将标识符存入 redis，如果处理过则直接返回。

- 大量消息在 mq 积压怎么解决？

通过手动扩容，将消费者资源扩大，快速消费积压数据，再恢复原本架构。

- mq 中消息过期怎么办？

这个场景比较特定，因为在使用 RabbitMQ 一般会设置 TTL，如果到了过期时间就进入到死信队列进行处理了。

- RabbitMQ 有几种广播类型，并讲讲作用？

有三种

1，Fanout 广播模式

向所有与这个交换机绑定的队列发消息

2，Direct 路由模式

消息发送时会带特定的路由键，路由交换机会根据路由键将消息路由到与之绑定且路由键完全匹配的队列。

3，Topic 主题模式

和路由模式基本相同，唯一不同的是，路由键可以包含通配符，“\*”代表一个单词，“#”表示 0 个或多个单词。
