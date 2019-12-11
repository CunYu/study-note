### 概述

线程组是用来管理线程的。

每个线程线程都归属于线程组，线程的默认线程组是main线程组，线程组也允许嵌套。

线程只能修改其所在线程组中其他线程的数据，其不能修改其他线程组中线程的数据。

### 示例

``` java
ThreadGroup threadGroup = new ThreadGroup("线程组");

Thread threadOne = new Thread(threadGroup, () -> {
    while (!Thread.currentThread().isInterrupted()) {
    }
    System.out.println("线程一停止");
}, "线程一");

Thread threadTwo = new Thread(threadGroup, () -> {
    while (!Thread.currentThread().isInterrupted()) {
    }
    System.out.println("线程二停止");
}, "线程二");

System.out.println(threadGroup.activeCount());

threadOne.start();
threadTwo.start();

System.out.println(threadGroup.activeCount());

Thread[] threads = new Thread[2];
threadGroup.enumerate(threads);

for (Thread thread : threads) {
    System.out.println(thread.getName());
}

threadGroup.interrupt();
```

``` text
0
2
线程一
线程二
线程一停止
线程二停止
```