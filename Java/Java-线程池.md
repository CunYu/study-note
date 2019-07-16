本文介绍了Java-线程池相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 概述

Java线程池是实际使用时比较常见的并发框架，其维护了多个线程在容器内以方便使用。

### 线程池优点

* 降低资源消耗

* 提高响应速度

* 方便管理线程

### 线程池原理

线程池维护了多个线程在容器中，当新的任务到线程池时，线程池会根据下面的逻辑去处理任务。

* 判断当前运行的线程是否小于corePoolSize，如果小于则创建新线程去处理任务，如果运行的线程等于或多于corePoolSize则将任务加入BlockingQueue。

* 如果BlockingQueue已满，则判断当前运行的线程是否超过了maximumPoolSize，如果没超过则新建线程执行任务，如果当前运行的线程已超过了maximumPoolSize则拒绝任务，并调用RejectedExecutionHandler.rejectedExecution()方法。

如果当前运行的线程小于corePoolSize，即使当前运行的线程有处于空闲状态的，其还是会新建线程去处理任务。

创建线程时会获取全局锁，获取全局锁会影响性能，应尽量避免获取全局锁。

线程池会将线程封装成Worker，Worker执行完当前任务后，会去BlockingQueue中获取新的任务执行。

### 线程池使用

线程池创建方式有两种，一种是自定义线程池，一种是通过Executors的静态方法来创建。

##### 自定义线程池

``` java
/**
 * 设置创建线程的工厂
 * 使用开源框架guava提供的ThreadFactoryBuilder可给线程池中的线程设置名字
 * compile("com.google.guava:guava:28.0-jre")
 */
ThreadFactory threadFactory = new ThreadFactoryBuilder().setNameFormat("demo-thread-%d").build();
/**
 * BlockingQueue<Runnable> workQueue是指工作队列
 * 常见工作队列有ArrayBlockingQueue，LinkedBlockingQueue，SynchronousQueue，PriorityBlockingQueue
 * ArrayBlockingQueue基于数组的有界阻塞队列，该队列按FIFO排序
 * LinkedBlockingQueue基于链表的有界阻塞队列，该队列按FIFO排序
 * SynchronousQueue不存储元素的阻塞队列，每个插入操作必须等到另一个线程进行移除操作
 * PriorityBlockingQueue具有优先级的无限阻塞队列，优先级高的任务先执行
 *
 * RejectedExecutionHandler handler是指饱和策略
 * AbortPolicy 抛出异常
 * CallerRunsPolicy 用调用者所在线程执行任务
 * DiscardOldestPolicy 丢弃队列中最老未处理的任务，然后重试
 * DiscardPolicy 不处理，直接丢弃任务
 *
 * long keepAliveTime 线程空闲后保持存活时间
 * TimeUnit unit 线程空闲后保持存活时间单位
 */
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(8, 8, 300, TimeUnit.SECONDS, new LinkedBlockingQueue<>(1024), threadFactory, new ThreadPoolExecutor.AbortPolicy());
/**
 * 调用该方法，线程池会在提前创建并启动所有核心线程
 */
threadPoolExecutor.prestartAllCoreThreads();

/**
 * 使用线程池执行任务有两种方式 submit和execute
 * submit用来执行需要返回值的任务
 * Future<T>是submit方法返回类型
 * future.get()获取任务返回值，该方法会阻塞当前线程
 * execute用来执行不需要返回值的任务
 */
IntStream.range(0, 300).forEach(i -> {
    threadPoolExecutor.execute(() -> {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ":" + i);
    });
});

/**
 * 关闭线程池有俩种方法shutdown和shutdownNow
 * 这两种方法会去遍历线程池中的所有线程并调用线程的interrupt方法，无法响应中断的方法将不会停止
 * shutdown会将线程池状态设置为SHUTDOWN状态，暂停接收任务并中断没有正在执行任务的线程
 * shutdownNow会将线程池状态设置为STOP装填，并中断线程池中的线程，并返回等待执行的任务列表
 * 只要调用了关闭线程池方法，isShutdown方法返回值就为true，但是这时线程池并不一定关闭
 * 线程池是否关闭可用isTerminaed方法判断
 */
threadPoolExecutor.shutdown();
System.out.println(threadPoolExecutor.isTerminated());
```

### 线程池配置

如果任务比较频繁且单个任务执行时间短可以适当提高keepAliveTime。

CPU密集型任务可以将线程数调小些，可参考CPU逻辑核心数+1。

IO密集型任务可以将线程数调高一些，可参考2倍CPU逻辑核心数。

建议使用有界队列，无界队列不容易及时发现问题，而且可能会造成队列中积累大量的任务，进而内存全被占用，有可能会引起一系列反应。