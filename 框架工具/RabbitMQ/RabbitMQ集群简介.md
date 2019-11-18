### RabbitMQ集群

单个RabbitMQ服务是承受不住大量数据的，而且也不可靠，RabbitMQ集群是必须的。

### RabbitMQ结构

<img src="./框架工具/RabbitMQ/image/RabbitMQ结构.png" alt="RabbitMQ结构"/>

### RabbitMQ元数据同步

RabbitMQ是用Erlang语言编写的，其本身具有同步的功能，所以其不需要其他工具来实现同步。

RabbitMQ为了能处理更多的数据，提高处理性能，其不会同步集群节点上的所有数据。

Queue中的数据不会在集群中各个节点存在，Queue数据只存在于创建其的结点上，其他节点只会保存该队列的元数据，如果需要将消息发往不是当前节点的队列中，当前节点会起到一个路由作用，其会将消息转发给对应队列所在的节点上。

##### 同步内容

* Exchange元数据

* Queue元数据

* 绑定元数据

* VHost元数据

### RabbitMQ节点

* Disk Node

Disk Node会将元数据保存在磁盘和内存中，一个RabbitMQ集群中至少有一个Disk Node。

* RAM Node

RAM Node仅将元数据保存在内存中。