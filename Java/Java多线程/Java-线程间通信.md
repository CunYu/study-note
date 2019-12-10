### volatile/synchronized

##### volatile

volatile是通过Lock前缀的指令实现的。

Lock前缀指令会将缓存中的数据回写到主内存并将其他缓存中对应的数据变为无效。

* 该关键字保证了其声明的变量在修改时对所有线程可见。

在操作系统中，每个线程都有自己专属的用来存储内存中部分数据的缓存，该缓存可以适当弥补CPU和内存之间读取速度的差异性，提高系统的性能。用volatile声明的变量，线程在修改时会将其改变刷新到内存中，并使其他线程对该变量的缓存无效，使其只能从内存中读取。但是过多的使用volatile会降低执行的效率。

* 该关键字禁止了指令的重新排序。

在Java中，为了提高性能，编译器和处理器常常会对指令重新排序。

编译器优化重排序，指令级并行重排序，内存系统重排序。

由于指令重排序的存在，会导致指令执行的顺序与实际代码有所出入，volatile可以禁止指令的重排序。

* 该关键字不保证原子性。

volatile只保证在读取volatile修饰的变量时读取的是最新的值。

##### synchronized

synchronized重量级锁，但自从Java SE 1.6起，对其进行了一定的优化，在一些情况下其变得不是那么重了。

synchronized可以使其所修饰的内容以同步的方式执行，其保证了其所修饰的内容同一时刻只能被同一个线程访问。

synchronized本质上是对对象的监视器（monitor）进行获取，同一时刻只能有一个线程获取，每个对象都有自己的monitor，同步代码块或同步方法执行时必须先获得monitor，没有获得monitor的会在同步块或同步方法的入口处阻塞，monitorenter和monitorexit指令实现了monitor的获取和释放，monitor存在于对象头中。

synchronized方法和synchronized(object)是一样的，以对象为锁。

静态synchronized方法和synchronized(Class)是一样的，都是以Class类为锁，Class锁对于该Class的所有实例都起效。

synchronized锁是可重入锁，自己可以再次获取自己已获得还未释放的锁，也支持父子类型（获取子类的锁也相当于获取其父类的锁）。

### 等待/通知机制

wait，notify，notifyAll。

wait方法会释放锁，而notify方法不会释放锁。

wait，notify，notifyAll只能在同步代码块中使用。

wait存在虚假唤醒的问题，也就是没有通过方法去唤醒wait，其会自己唤醒，所以使用时，要做虚假判断，如果唤醒条件不满足，则一直wait。

``` java
public class Wait implements Runnable {

    @Override
    public void run() {

        synchronized (Demo.lock) {

            while (!Demo.sw) {
                try {
                    System.out.println("等待线程等待中！");
                    Demo.lock.wait(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("等待线程运行完毕！");
        }
    }
}
```

``` java
public class Notify implements Runnable {

    @Override
    public void run() {

        synchronized (Demo.lock) {
            try {
                Thread.sleep(5000);
                Demo.sw = true;
                Demo.lock.notify();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

``` java
public class Demo {

    public static Object lock = new Object();
    public static volatile Boolean sw = false;

    public static void main(String[] args) throws InterruptedException {

        Thread wait = new Thread(new Wait());
        wait.start();
        Thread.sleep(5000);
        Thread notify = new Thread(new Notify());
        notify.start();
    }
}
```

运行结果

``` text
等待线程等待中！
等待线程等待中！
等待线程等待中！
等待线程等待中！
等待线程等待中！
等待线程运行完毕！
```

### 管道输入/输出流

Piped管道流以内存为媒介实现了线程之间的通信。

##### 字节

PipedOutputStream

PipedInputStream

##### 字符

PipedReader

PipedWriter

``` java
public class Producer implements Runnable {

    PipedOutputStream pipedOutputStream;

    public Producer(PipedOutputStream pipedOutputStream) {
        this.pipedOutputStream = pipedOutputStream;
    }

    @Override
    public void run() {

        try {
            System.out.println("发送者开始发送！");
            String message = "Hello World!";
            pipedOutputStream.write(message.getBytes());
            System.out.println("发送第一条信息结束！");
            Thread.sleep(5000);
            pipedOutputStream.write(message.getBytes());
            System.out.println();
            System.out.println("发送第二条信息结束！");
            pipedOutputStream.flush();
            pipedOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

``` java
public class Receiver implements Runnable {

    PipedInputStream pipedInputStream;

    public Receiver(PipedInputStream pipedInputStream) {
        this.pipedInputStream = pipedInputStream;
    }

    @Override
    public void run() {

        try {
            int flag;
            while (-1 != (flag = pipedInputStream.read())) {
                System.out.print((char) flag);
            }
            System.out.println();
            System.out.println("接收信息结束！");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

``` java
try {
    PipedOutputStream pipedOutputStream = new PipedOutputStream();
    PipedInputStream pipedInputStream = new PipedInputStream();
    pipedOutputStream.connect(pipedInputStream);

    Thread producer = new Thread(new Producer(pipedOutputStream));
    Thread receiver = new Thread(new Receiver(pipedInputStream));
    producer.start();
    receiver.start();
} catch (IOException e) {
    e.printStackTrace();
}
```

运行结果

``` text
发送者开始发送！
发送第一条信息结束！
Hello World!
发送第二条信息结束！
Hello World!
接收信息结束！
```

### Thread.join

表示当前线程要等待加入的线程终止后再开始执行。

其本身通过wait，notifyAll实现。

``` java
public class Join implements Runnable {

    @Override
    public void run() {

        try {
            System.out.println("Join开始执行！");
            Thread.sleep(5000);
            System.out.println("Join执行结束！");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

``` java
try {
    System.out.println("主线程开始运行！");
    Thread thread = new Thread(new Join());
    thread.start();
    thread.join();
    System.out.println("主线程运行结束！");
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

运行结果

``` text
主线程开始运行！
Join开始执行！
Join执行结束！
主线程运行结束！
```

### ThreadLocal

ThreadLocal，线程本地变量，以ThreadLocal对象为键，以对象为值的存储结构。线程之间不共享ThreadLocal存储的值。

``` java
public class Job implements Runnable {

    private ThreadLocal<Integer> threadLocal = new ThreadLocal<>();

    @Override
    public void run() {
        int randomNum = (int) (Math.random() * 1000);
        threadLocal.set(1 + randomNum);
        System.out.println("i初始值为1，随机数为" + randomNum + "，结束值为" + threadLocal.get().toString());
        // 避免内存泄漏
        threadLocal.remove();
    }
}
```

``` java
Thread threadOne = new Thread(new Job());
Thread threadTwo = new Thread(new Job());
threadOne.start();
threadTwo.start();
```

运行结果

``` text
i初始值为1，随机数为440，结束值为441
i初始值为1，随机数为282，结束值为283
```