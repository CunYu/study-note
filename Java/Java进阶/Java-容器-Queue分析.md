### Queue

Queue是一个FIFO（先入先出）的数据结构，其继承了Collection接口，其可以用数据或链表实现。

### Deque

Deque继承了Queue，其是一个双向队列，Queue只能是一个方向的，一边进入数据，一边出数据，而Deque是双向队列，其两边都可以进出数据。

Deque可以用来替换Stack。

### LinkedList

LinkedList实现了Deque。

### PriorityQueue

优先队列，其基于堆实现，其每次取值优先取最小元素，其最小值的判断是根据自然顺序，也可以使用指定的排序器比较。

### BlockingQueue

阻塞队列，其会在特定情况下阻塞操作队列的线程。

### ConcurrentLinkedQueue

线程安全的基于链表的Queue。

### ConcurrentLinkedDeque

线程安全的基于链表的Deque。