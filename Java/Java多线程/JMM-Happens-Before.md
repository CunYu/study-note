### 概述

JMM中使用happens-before来表示两个操作之间的执行顺序。

如果A操作happens-before B操作，那么A操作一定在B操作之前，A操作结果对B操作可见，指令重排序也不能违背happens-before。

### 规则

* 程序顺序

单线程中的每个操作都happens-before其后续操作。

* 监视器锁

解锁happens-before加锁。

* volatile

对一个volatile变量的写happens-before对volatile变量的读。

* 传递性

A操作happens-before B操作，B操作happens-before C操作，那么A操作happens-before C操作。

* 线程启动

线程的start操作hppens-before线程中的其他任意操作。

* join

join线程的所有操作happens-before其join方法返回操作。