### 线程进程

##### 进程

进程是现代操作系统调度和资源分配的基本单位，现代操作系统会为其每个运行中的程序创建一个进程。

##### 线程

线程是现代操作系统调度的最小单位，也称轻量级进程，线程不拥有资源，线程会归属于某一进程，同一进程下的各个线程会共用进程的资源。

### 线程优先级

线程优先级是用来决定线程获取处理器资源的优先级。

线程的优先级不是绝对的，而且有些操作系统会忽略线程的优先级设置。

线程的优先级默认为5，范围为1~10。

|优先级常量|含义|
|:----|:----|
|Thread.MIN_PRIORITY|1|
|Thread.NORM_PRIORITY|5|
|Thread.MAX_PRIORITY|10|

线程可通过setPriority方法设置优先级。

### 守护（Daemon）线程

守护线程是支持性线程，常用来做后台调度，系统支持等工作。

在JVM中，如果不存在非守护线程，JVM将会退出。

在线程启动前，可通过setDaemon方法将其设置为守护线程。

### 初始化线程

##### 继承Thread

``` java
public class Job extends Thread {

    @Override
    public void run() {
        System.out.println("线程运行中！");
    }
}
```

``` java
Job job = new Job();
job.start();
```

##### 实现Runnable

``` java
public class Job implements Runnable {

    @Override
    public void run() {
        System.out.println("线程运行中！");
    }
}
```

``` java
Thread thread = new Thread(new Job());
thread.start();
```

线程创建时，为方便管理，可通过传参对线程命名。

##### 实现Callable

``` java
public class Job implements Callable<String> {

    @Override
    public String call() {
        return "Hello World!";
    }
}
```

``` java
FutureTask<String> futureTask = new FutureTask<>(new Job());
Thread thread = new Thread(futureTask);
thread.start();
System.out.println(futureTask.get());
```

### 线程状态

|线程状态|说明|
|:----|:----|
|NEW|表示线程已构建，但是还没调用start方法。|
|RUNNABLE|表示线程已就绪等待获取获取CPU资源中或线程已获取到CPU资源正在运行。|
|BLOCKED|表示线程被锁阻塞。|
|WAITING|表示线程正在等待。|
|TIMED_WAITING|表示线程正在超时等待。|
|TERMINATED|表示线程已执行完毕|

状态枚举：Thread.State

Java将操作系统中的运行和就绪状态统称为运行状态。

### 线程异常

线程的异常尽量在线程中处理，通过线程的setUncaughtExceptionHandler方法可以设置线程抛出异常的处理方法。

### yield

线程的yield方法可以使线程放弃当前获取的时间片，由运行中状态变为可运行状态，该方法的具体效果取决于操作系统的调度，有可能刚放弃获取的时间片又重新获得时间片。

### 线程相关概念

##### 同步异步

同步指的是发起者必须等到执行结果才继续执行，异步则不关心执行结果。

同步异步是站在生产者的角度。

##### 并发并行

并发不是各个任务同时执行，其指的是一个时间段内各个任务交替执行。

并行指的是各个任务同时执行。

##### 阻塞非阻塞

阻塞指的是如果不能得到结果，则不往下执行，当前线程会被挂起，同步情况下容易引起阻塞。

非阻塞指的是即使得不到结果，也会往下执行，不会阻塞当前线程。

阻塞非阻塞是站在消费者的角度。

##### 临界区

多个线程共享资源，同一时刻只能被一个线程访问，一旦临界区资源被一个线程访问，在该线程释放资源之前，其不能被其他线程访问。

##### 上下文切换

线程在切换前会保存当前环境，当下次切换回来时，会根据保存的环境进行恢复。线程保存环境，恢复环境就称为一次上下文切换。

### 安全的终止线程

线程的suspend，resume，stop方法由于在资源处理上的不足，已不建议使用。

我们可以通过标识符或中断操作的方式来终止线程并使其资源得到有效的释放。

``` java
public class Job implements Runnable {

    @Override
    public void run() {
        while (Demo.run && !Thread.currentThread().isInterrupted()) {
            System.out.println("线程运行中！");
        }
    }
}
```

``` java
public class Demo {

    public static volatile boolean run = true;

    public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(new Job());
        thread.start();
        Thread.sleep(5000);
        run = false;
        //thread.interrupt();
    }
}
```