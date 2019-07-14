本文介绍了Java-队列同步器相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 概述

队列同步器（AbstractQueuedSynchronizer，AQS）是用来构建锁或其它同步组件的基础框架。

队列同步器是实现锁或其它同步组件的关键。

### 队列同步器结构

队列同步器使用一个int成员变量表示同步状态，使用一个FIFO双向队列来表示同步队列，同步器会存贮同步状态，同步队列首尾节点。

##### 同步队列

同步队列中Node节点用来保存获取同步状态失败的线程引用。

* Node节点主要属性

|名称|说明|
|:----|:----|
|waitStatus|等待状态|
|prev|前驱节点|
|next|后继节点|
|nextWaiter|下一个等待节点或者特殊的SHARED节点|
|thread|节点存储的线程引用|

节点进入同步队列中，其就进入了自旋的状态，其会一直无限循环的判断条件，当条件满足，获取到了同步状态，其就会退出自旋。

### 队列同步器抽象类

队列同步器主要是通过继承AbstractQueuedSynchronizer抽象类来使用，其使用了模板方法模式，我们只需要重写指定的方法便可以实现同步状态的管理。

##### 同步状态方法

|方法名|说明|
|:----|:----|
|getState|获取同步状态|
|setState|设置同步状态|
|compareAndSetState|通过CAS来设置同步状态|

##### 子类重写主要方法

|方法名|说明|
|:----|:----|
|tryAcquire|独占式获取同步状态|
|tryRelease|独占式释放同步状态|
|tryAcquireShared|共享式获取同步状态|
|tryReleaseShared|共享式释放同步状态|
|isHeldExclusively|判断同步状态是否被当前线程独占|

##### 模板方法

|方法名|说明|
|:----|:----|
|acquire|独占式获取同步状态|
|acquireInterruptibly|可中断独占式获取同步状态|
|tryAcquireNanos|超时独占式获取同步状态|
|release|独占式释放同步状态|
|acquireShared|共享式获取同步状态|
|acquireSharedInterruptibly|可中断共享式获取同步状态|
|tryAcquireSharedNanos|超时共享式获取同步状态|
|releaseShared|共享式释放同步状态|
|getQueuedThreads|获取等待在同步状态上的线程队列集合|

### 自定义同步组件

MyLock

``` java
public class MyLock implements Lock {

    private static final class Sync extends AbstractQueuedSynchronizer {

        @Override
        protected boolean tryAcquire(int arg) {

            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int arg) {
            if (compareAndSetState(1, 0)) {
                setExclusiveOwnerThread(null);
                return true;
            }
            return false;
        }

        Condition newCondition() {
            return new ConditionObject();
        }
    }

    private Sync sync = new Sync();

    @Override
    public void lock() {
        sync.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.release(1);
    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

Job

``` java
public class Job implements Runnable {

    private MyLock lock = new MyLock();

    @Override
    public void run() {

        lock.lock();
        try {
            Thread.sleep(3000);
            IntStream.range(0, 3).forEach(i -> {
                System.out.println(Thread.currentThread() + "第" + i + "次执行！");
            });
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

主程序

``` java
Job job = new Job();
Thread threadOne = new Thread(job, "threadOne");
Thread threadTwo = new Thread(job, "threadTwo");
Thread threadThree = new Thread(job, "threadThree");
Thread threadFour = new Thread(job, "threadFour");
Thread threadFive = new Thread(job, "threadFive");

threadOne.start();
threadTwo.start();
threadThree.start();
threadFour.start();
threadFive.start();
```

输出

``` text
Thread[threadOne,5,main]第0次执行！
Thread[threadOne,5,main]第1次执行！
Thread[threadOne,5,main]第2次执行！
Thread[threadThree,5,main]第0次执行！
Thread[threadThree,5,main]第1次执行！
Thread[threadThree,5,main]第2次执行！
Thread[threadTwo,5,main]第0次执行！
Thread[threadTwo,5,main]第1次执行！
Thread[threadTwo,5,main]第2次执行！
Thread[threadFive,5,main]第0次执行！
Thread[threadFive,5,main]第1次执行！
Thread[threadFive,5,main]第2次执行！
Thread[threadFour,5,main]第0次执行！
Thread[threadFour,5,main]第1次执行！
Thread[threadFour,5,main]第2次执行！
```