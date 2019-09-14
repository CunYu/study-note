### Lock

锁是控制多个线程访问共享资源的工具。

在Lock接口出现前（Lock出现在java se 1.5），java通过synchronized来实现锁功能。

Lock在操作上比synchronized更灵活，功能上也相对synchronized多。

##### 锁的使用注意

``` java
Lock lock = new ReentrantLock();
lock.lock();
try {
    System.out.println("Hello World!");
} catch (Exception e) {
    e.printStackTrace();
} finally {
    lock.unlock();
}
```

使用Lock的时候，要千万考虑到锁的释放，以避免出现无法释放资源的情况。synchronized发生异常时也会释放锁。

##### 锁常用方法

|方法|说明|
|:----|:----|
|void lock()|获取锁，如果没有获取锁，该线程不会接受线程调度，并且休眠直到获得锁。|
|void lockInterruptibly() throws InterruptedException|可中断获取锁，和lock不同之处在于其在获取锁的过程中可以被中断。|
|boolean tryLock()|尝试非阻塞的获取锁，该方法会立即返回，如果获取到锁则返回true，否则则为false。|
|boolean tryLock(long time, TimeUnit unit) throws InterruptedException|超时获取锁，该方法在3种情况下返回：1，指定时间内获得了锁，返回true；2，获取锁的线程在超时时间内被中断，返回false；3，过了超时时间还没获得锁，放回false。|
|void unlock()|释放锁|
|Condition newCondition()|获取与其绑定的等待通知组件。|

### 锁的相关知识

##### 可中断锁

在获取过程中线程可以被中断的锁，synchronized是不可中断锁，Lock是可中断锁。

##### 公平锁

公平锁尽量保证按线程获取锁的顺序分配锁，当一个锁释放时，等待最久的线程优先获取锁。

非公平锁不保证按线程获取锁的顺序分配锁，这种情况下，有些线程有可能会永远获取不到锁。

公平锁为了保证公平付出了很大的性能代价，非公平锁虽然有可能导致一些线程永远获取不到锁，但是其开销小。

##### 排他锁

同一时刻只能一个线程访问。

### 常见锁

ReentrantLock，ReentrantReadWriteLock。

### ReentrantLock

重入锁，当前线程在获取到锁后可以再次获得该锁。该锁是排他锁。

synchronized是重入锁。

``` java
public class Job implements Runnable {

    private Lock lock = new ReentrantLock();

    @Override
    public void run() {
        lock.lock();
        try {
            Thread.sleep(5000);
            IntStream.range(0, 5).forEach(i -> {
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

``` java
Job job = new Job();
Thread threadA = new Thread(job, "A");
Thread threadB = new Thread(job, "B");
Thread threadC = new Thread(job, "C");
Thread threadD = new Thread(job, "D");

threadA.start();
threadB.start();
threadC.start();
threadD.start();
```

运行结果

``` text
Thread[A,5,main]第0次执行！
Thread[A,5,main]第1次执行！
Thread[A,5,main]第2次执行！
Thread[A,5,main]第3次执行！
Thread[A,5,main]第4次执行！
Thread[D,5,main]第0次执行！
Thread[D,5,main]第1次执行！
Thread[D,5,main]第2次执行！
Thread[D,5,main]第3次执行！
Thread[D,5,main]第4次执行！
Thread[B,5,main]第0次执行！
Thread[B,5,main]第1次执行！
Thread[B,5,main]第2次执行！
Thread[B,5,main]第3次执行！
Thread[B,5,main]第4次执行！
Thread[C,5,main]第0次执行！
Thread[C,5,main]第1次执行！
Thread[C,5,main]第2次执行！
Thread[C,5,main]第3次执行！
Thread[C,5,main]第4次执行！
```

### ReentrantReadWriteLock

读写锁，非排他锁，其自身维护了两个锁，一个读锁，一个写锁，读锁之间不互斥，写锁与读，写锁都互斥，写锁是可重入的排它锁。

##### 读锁示例

``` java
public class Job implements Runnable {

    private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    @Override
    public void run() {
        lock.readLock().lock();
        try {
            Thread.sleep(5000);
            IntStream.range(0, 5).forEach(i -> {
                System.out.println(Thread.currentThread() + "第" + i + "次执行！");
            });
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.readLock().unlock();
        }
    }
}
```

``` java
Job job = new Job();
Thread threadA = new Thread(job, "A");
Thread threadB = new Thread(job, "B");
Thread threadC = new Thread(job, "C");
Thread threadD = new Thread(job, "D");

threadA.start();
threadB.start();
threadC.start();
threadD.start();
```

运行结果

``` text
Thread[A,5,main]第0次执行！
Thread[B,5,main]第0次执行！
Thread[C,5,main]第0次执行！
Thread[D,5,main]第0次执行！
Thread[C,5,main]第1次执行！
Thread[B,5,main]第1次执行！
Thread[A,5,main]第1次执行！
Thread[B,5,main]第2次执行！
Thread[C,5,main]第2次执行！
Thread[D,5,main]第1次执行！
Thread[C,5,main]第3次执行！
Thread[B,5,main]第3次执行！
Thread[A,5,main]第2次执行！
Thread[B,5,main]第4次执行！
Thread[C,5,main]第4次执行！
Thread[D,5,main]第2次执行！
Thread[A,5,main]第3次执行！
Thread[D,5,main]第3次执行！
Thread[A,5,main]第4次执行！
Thread[D,5,main]第4次执行！
```

##### 写锁示例

``` java
public class Job implements Runnable {

    private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    @Override
    public void run() {
        lock.writeLock().lock();
        try {
            Thread.sleep(5000);
            IntStream.range(0, 5).forEach(i -> {
                System.out.println(Thread.currentThread() + "第" + i + "次执行！");
            });
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

``` java
Job job = new Job();
Thread threadA = new Thread(job, "A");
Thread threadB = new Thread(job, "B");
Thread threadC = new Thread(job, "C");
Thread threadD = new Thread(job, "D");

threadA.start();
threadB.start();
threadC.start();
threadD.start();
```

运行结果

``` text
Thread[B,5,main]第0次执行！
Thread[B,5,main]第1次执行！
Thread[B,5,main]第2次执行！
Thread[B,5,main]第3次执行！
Thread[B,5,main]第4次执行！
Thread[C,5,main]第0次执行！
Thread[C,5,main]第1次执行！
Thread[C,5,main]第2次执行！
Thread[C,5,main]第3次执行！
Thread[C,5,main]第4次执行！
Thread[A,5,main]第0次执行！
Thread[A,5,main]第1次执行！
Thread[A,5,main]第2次执行！
Thread[A,5,main]第3次执行！
Thread[A,5,main]第4次执行！
Thread[D,5,main]第0次执行！
Thread[D,5,main]第1次执行！
Thread[D,5,main]第2次执行！
Thread[D,5,main]第3次执行！
Thread[D,5,main]第4次执行！
```

##### 读写混合示例

``` java
public class Content {

    private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    private String value;

    public Content(String value) {
        this.value = value;
    }

    public void read() {
        lock.readLock().lock();
        try {
            System.out.println(value);
        } finally {
            lock.readLock().unlock();
        }
    }

    public void write(String value) {
        lock.writeLock().lock();
        try {
            System.out.println("开始写入！");
            Thread.sleep(5000);
            this.value = value;
            System.out.println("写入完毕！");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

``` java
public class Read implements Runnable {

    private Content content;

    public Read(Content content) {
        this.content = content;
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 5; i++) {
                content.read();
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

``` java
public class Write implements Runnable {

    private Content content;

    public Write(Content content) {
        this.content = content;
    }

    @Override
    public void run() {
        try {
            Thread.sleep(1000);
            content.write("Hello World!");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

``` java
Content content = new Content("Hello");

Thread threadA = new Thread(new Read(content), "A");
Thread threadB = new Thread(new Read(content), "B");
Thread threadC = new Thread(new Read(content), "C");
Thread threadD = new Thread(new Write(content), "D");

threadA.start();
threadB.start();
threadC.start();
threadD.start();
```

运行结果

``` text
Hello
Hello
Hello
Hello
Hello
Hello
开始写入！
写入完毕！
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
```

##### 锁降级

读写锁支持锁降级，不支持锁升级。

锁降级是指当前已获取写锁，在不释放写锁的情况下，再去获取读锁，然后释放写锁的过程。

``` java
public class Job implements Runnable {

    private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    @Override
    public void run() {
        lock.writeLock().lock();
        try {
            Thread.sleep(5000);
            lock.readLock().lock();
            IntStream.range(0, 5).forEach(i -> {
                System.out.println(Thread.currentThread() + "第" + i + "次执行！");
            });
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.writeLock().unlock();
            lock.readLock().unlock();
        }
    }
}
```

### LockSupport

LockSupport是用来实现线程的阻塞与唤醒。

* LockSupport使用方便，灵活，而wait，notify，notifyAll只能在同步代码块中使用。

* park和wait一样会释放锁，一样使线程进入等待状态，一样的可以用interrupt中断等待，不过wait会抛出中断异常，而part不会。

* LockSupport中唤醒可以优先阻塞使用。

* 已处于park状态，如再次调用park方法会导致无法唤醒。

示例

``` java
public class Park implements Runnable {

    @Override
    public void run() {

        try {
            Thread.sleep(5000);
            System.out.println(Thread.currentThread() + "park");
            LockSupport.park();
            System.out.println(Thread.currentThread() + "unpark");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

``` java
Thread threadA = new Thread(new Park(), "A");
threadA.start();
LockSupport.unpark(threadA);
System.out.println(Thread.currentThread() + "unpack");
```

运行结果

``` text
Thread[main,5,main]unpack
Thread[A,5,main]park
Thread[A,5,main]unpark
```

### Condition

Condition的出现可以使我们更好的实现线程的等待/唤醒，其比wait，notify更灵活，更精细。

Condition可以通过锁来构建，其可以构建多个Condition对象，signal方法只是将对应Condition上的等待的线程唤起。

wait，notify相当于只有一个Condition。

示例

``` java
public class Test {

    private Lock lock = new ReentrantLock();
    public Condition conditionA = lock.newCondition();
    public Condition conditionB = lock.newCondition();

    public void awaitConditionAOne() {
        try {
            lock.lock();
            System.out.println("awaitConditionAOne-begin");
            conditionA.await();
            System.out.println("awaitConditionAOne-end");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void awaitConditionATwo() {
        try {
            lock.lock();
            System.out.println("awaitConditionATwo-begin");
            conditionA.await();
            System.out.println("awaitConditionATwo-end");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void awaitConditionB() {
        try {
            lock.lock();
            System.out.println("awaitConditionB-begin");
            conditionB.await();
            System.out.println("awaitConditionB-end");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void signalAllConditionA() {
        try {
            lock.lock();
            System.out.println("signalAllConditionA-begin");
            conditionA.signalAll();
            System.out.println("signalAllConditionA-end");
        } finally {
            lock.unlock();
        }
    }

    public void signalAllConditionB() {
        try {
            lock.lock();
            System.out.println("signalAllConditionB-begin");
            conditionB.signalAll();
            System.out.println("signalAllConditionB-end");
        } finally {
            lock.unlock();
        }
    }
}
```

``` java
public class AwaitConditionAOne implements Runnable {

    private Test test;

    public AwaitConditionAOne(Test test) {
        this.test = test;
    }

    @Override
    public void run() {
        test.awaitConditionAOne();
    }
}
```

``` java
public class AwaitConditionATwo implements Runnable {

    private Test test;

    public AwaitConditionATwo(Test test) {
        this.test = test;
    }

    @Override
    public void run() {
        test.awaitConditionATwo();
    }
}
```

``` java
public class AwaitConditionB implements Runnable {

    private Test test;

    public AwaitConditionB(Test test) {
        this.test = test;
    }

    @Override
    public void run() {
        test.awaitConditionB();
    }
}
```

``` java
public class SignalAllConditionA implements Runnable {

    private Test test;

    public SignalAllConditionA(Test test) {
        this.test = test;
    }

    @Override
    public void run() {
        test.signalAllConditionA();
    }
}
```

``` java
public class SignalAllConditionB implements Runnable {

    private Test test;

    public SignalAllConditionB(Test test) {
        this.test = test;
    }

    @Override
    public void run() {
        test.signalAllConditionB();
    }
}
```

``` java
Test test = new Test();

Thread awaitConditionAOne = new Thread(new AwaitConditionAOne(test));
Thread awaitConditionATwo = new Thread(new AwaitConditionATwo(test));
Thread awaitConditionB = new Thread(new AwaitConditionB(test));
Thread signalAllConditionA = new Thread(new SignalAllConditionA(test));
Thread signalAllConditionB = new Thread(new SignalAllConditionB(test));

awaitConditionAOne.start();
awaitConditionATwo.start();
awaitConditionB.start();
Thread.sleep(5000);
signalAllConditionA.start();
Thread.sleep(5000);
signalAllConditionB.start();
```

运行结果

``` text
awaitConditionAOne-begin
awaitConditionB-begin
awaitConditionATwo-begin
signalAllConditionA-begin
signalAllConditionA-end
awaitConditionAOne-end
awaitConditionATwo-end
signalAllConditionB-begin
signalAllConditionB-end
awaitConditionB-end
```