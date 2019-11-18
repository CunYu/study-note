### RabbitMQ 简述

RabbitMQ是一个实现AMQP的消息中间件，其服务器端使用Erlang语言编写。

### RabbitMQ 流程图

<img src="./框架工具/RabbitMQ/image/RabbitMQ流程图.png" alt="RabbitMQ流程图"/>

Exchange和Queue存在于服务端上，Producer和Consumer分别存在于客户端上。

### RabbitMQ 通信机制

RabbitMQ是基于TCP的Channel来通信的，Channel是存在与TCP连接内的虚拟连接，每个TCP连接内的Channel数量是不受限制的。

### RabbitMQ 基本概念

##### Producer

生产消息的服务。

##### Exchange

* 交换路由，将Producer发过来的信息转发给对应的Queue，其不提供存储消息的功能，如没有对应的Queue，则丢失消息。

* Exchange与Queue的关联操作叫绑定。

##### RouteKey

* RouteKey在Producer发送消息至Exchange时指定，Exchange会根据RouteKey去匹配对应Queue。

* RouteKey不是必须指定的。

##### Exchange 类型

* fanout

不需要RouteKey，将消息广发到所有与Exchange绑定的Queue中。

* direct

将信息分发给所有与Exchange绑定的，routingKey对应一致的队列。

默认Exchange为direct类型，默认Exchange的RouteKey为队列名。

* topic

将信息分发给所有与Exchange绑定的，routingKey按规则匹配的队列。

routingKey为用 . 分割的一连串标识符，最大长度为255个字节。

匹配规则中，\* 表示一个标识符，# 表示0个，1个或多个标识符。

* headers

不使用routingKey，而是通过消息头设置键值对进行匹配处理，会将信息分发给所有与Exchange绑定的，消息头匹配的队列。

消息头键值对中，x-match = all 表示所有的键值对必须都匹配，x-match = any 表示只要有键值对匹配即可。

##### Queue

* 消息存储的地方。

* 队列被多个Consumer消费时，会均摊到Consumer上。

* 可以设置Consumer的最大同时消费数。

* 临时队列，一旦消费者断开，队列就自动删除了，非持久的，唯一的。

##### Consumer

消费消息的服务。

##### Durability

* 为了防止RabbitMQ意外中断消息丢失，可以将消息和队列设置为持久化（将消息和队列保存在硬盘中），这样可以保证大多数的消息不会丢失。

* 消息和队列的持久化不能保证所有信息都不会丢失，当rabbitmq接收到消息还未来得及持久化，此时服务中断，消息是会丢失的。

### 消息确认

RabbitMQ存在消息确认机制，其用来保证消息已被正确的处理。

##### Confirm 模式

Producer可以将Channel设置为Confirm模式，所有在Channel发送的消息会用唯一的ID标识，当消息发往队列中后，会告知Producer，这个操作是异步的。

如果Producer没有收到确认消息（无确认消息，确认失败），其会重新发送消息，Queue会有幂等拦截，相同ID标识的数据不会出现在Queue中。

##### Message acknowledgment

* Consumer用来返回给RabbitMQ的确认消息来表示消息已被Consumer接收，处理，RabbitMQ可以从Queue中删除对应的消息了。

* Message acknowledgment可以设置自动返回和手动返回。

* Message acknowledgment不返回时，会导致队列中消息越来越多。

### VHost

VHost是RabbitMQ服务中的虚拟主机，RabbitMQ服务中的VHost起到了隔离的作用，其相当于一个虚拟的，独立的RabbitMQ服务存在RabbitMQ服务中。

### 死信队列

DLX（Dead Letter Exchange）,当消息变为死信后，其会被发往到DLX，DLX是一个正常的Exchange。

##### 死信条件

* 消息被拒绝

* 消息TTL过期

* 队列已满

### Java Demo 地址

[https://github.com/CunYu/rabbitmq-java-demo](https://github.com/CunYu/rabbitmq-java-demo)