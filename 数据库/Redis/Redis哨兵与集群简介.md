### 哨兵

Sentinel，哨兵模式正如其名，是用来监控Redis的。

单机Redis并不适合生产使用，所以Redis支持主从复制，其可以有一个主Redis和一个或多个从Redis，主Redis支持读写，从Redis只支持支持读，从Redis会从主Redis复制数据。

哨兵类似服务发现组件，其是用来监控主从Redis的，当主Redis停止服务时，其会从从Reids中选举一个作为新的主Redis。

程序使用时，是通过哨兵获取主Redis地址的。

哨兵可以组成集群，哨兵之间也会互相监控。

##### 主从复制

* 全量同步

从Redis会向主Redis发起同步请求，主Redis收到请求后，会执行RDB操作，并用缓存区缓存后续写操作命令。

主Redis执行完RDB操作后，会将RDB快照文件发往从Redis。

从Redis收到RDB快照文件后，会丢弃自己所有数据，载入收到的RDB快照文件。

从Redis载入RDB快照文件后，开始接受并执行主Redis缓存区的写操作命令。

* 增量同步

增量同步是指主Redis每执行一个写操作都会将写操作命令发给从Redis执行。

* 策略

从Redis往往在第一次连接主Redis时采用全量同步，全量同步后执行增量同步。

Redis会优先选择增量同步，增量同步失败后再进行全量同步。

### 集群

单个Redis服务并发能力有限，而且存储也有限，Redis支持集群。

Redis使用哈希槽的概念，Redis将哈希槽分为16384个，集群中的每个节点负责一部分槽。

当数据需要存储时，其会先使用CRC16处理，然后用处理后的值对16384取模来确定其存储在那个槽。

### Redis服务结构

一个可靠的Redis服务应该最少有三个主节点，三个从节点，三个哨兵。