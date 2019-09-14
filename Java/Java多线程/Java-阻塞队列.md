### 概述

阻塞队列（BlockingQueue）是一个支持阻塞操作的队列。

* 阻塞插入操作

当队列已满时，队列会阻塞执行插入操作的线程，直到队列不满。

* 阻塞移除操作

当队列为空时，队列会阻塞执行移除操作的线程，直到队列非空。

### 阻塞队列基本操作比较

|操作|抛出异常|返回特殊值|阻塞|先阻塞，超出指定的时间则退出|
|:----|:----|:----|:----|:----|
|队列已满插入|add(e)|offer(e)|put(e)|offer(e, timeout, unit)|
|队列为空移除|remove|poll|take|offer|

### 常见阻塞队列

|名称|说明|
|:----|:----|
|ArrayBlockingQueue|数据结构的有界阻塞队列|
|LinkedBlockingQueue|链表结构的有界阻塞队列|
|PriorityBlockingQueue|支持优先级排序的无界阻塞队列|
|DelayQueue|使用优先级队列实现的无界阻塞队列|
|SynchronousQueue|不存储元素的阻塞队列|
|LinkedTransferQueue|链表结构的无界阻塞队列|
|LinkedBlockingDeque|链表结构的双向阻塞队列|

### 阻塞队列实现原理

阻塞队列通过运用Condition来区分队列不同的情况，使用await方法来使线程阻塞，使用signal方法来使线程唤醒。