### 概述

Kafka是一个分布式流处理平台，其现归属于Apache，是用Scala和Java编写的。

##### 分布式流处理平台特性

* 支持发布和订阅的消息队列

* 存储，处理流式记录

### 结构

<img src="./框架工具/Kafka/image/Kafka结构.png" alt="Kafka结构"/>

##### Producer

Producer是消息生产者，其往Kafka发送消息。

发送消息需要指明topic，如果传递了partition信息，那么会发往指定的partition，如果传递了key信息，那么会根据key的哈希来确定分区，如果都没有传，则循环发往各个分区。

发送信息并不是一条一条的发往Kafka，其会积攒一批消息，按批发送给Kafka。

##### Topic

Kafka使用Topic来对消息进行分类，发布和订阅是以Topic为单位的。

##### Partition

Topic使用分区来存储消息，kafka Cluster中的每个Borker都会有分区文件，其会选取一个leader分区，其他的分区为follower分区，leader分区负责处理消息，follower分区负责从leader同步消息。

Partition是以队列的形式来存储消息的，其可以持久化，也可以指定数据存储的时间。

##### Consumer Group

Consumer Group会从其订阅的Topic获取数据。

Consumer Group是由一个或多个Consumer组成的，同一个Consumer Group中的两个Consumer不得消费同一个分区，也就是一个分区的消息只能被Consumer Group中的一个Consumer消费。

Consumer会记录其消费分区的offset，offsct表示当前消费者消费到了分区的那一条消息，offset可以调整。