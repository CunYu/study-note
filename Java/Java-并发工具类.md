本文介绍了Java并发工具类相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### CountDownLatch

CountDownLatch用来等待其他线程完成操作，可以实现join的功能，并且比join更加灵活。

##### 示例

``` java
CountDownLatch countDownLatch = new CountDownLatch(1);

new Thread(() -> {
    try {
        Thread.sleep(10000);
        System.out.println("线程执行完毕");
        countDownLatch.countDown();
    } catch (Exception e) {
        e.printStackTrace();
    }
}).start();

countDownLatch.await();
System.out.println("Hello World");
```

``` text
线程执行完毕
Hello World
```

##### 分析

CountDownLatch在初始化的时候会传入一个值，当这个值为0时，其await方法不会阻塞当前线程，当这个值不为0时，其await方法会阻塞当前线程，其countDown方法会使该值减1。

CountDownLatch中的值不可重新设置，一个CountDownLatch只能使用一次。

### CyclicBarrier

同步屏障CyclicBarrier可以使线程阻塞，直到阻塞的线程数量到达预设值后才解除阻塞。

##### 示例

``` java
CyclicBarrier cyclicBarrier = new CyclicBarrier(2);

new Thread(() -> {
    try {
        Thread.sleep(10000);
        cyclicBarrier.await();
        System.out.println("线程执行完毕");
    } catch (Exception e) {
        e.printStackTrace();
    }
}).start();

cyclicBarrier.await();
System.out.println("Hello World");
```

``` text
线程执行完毕
Hello World
```

##### 分析

CyclicBarrier在初始化时会传入一个值，当其await的线程达到这个值后，才唤醒其阻塞的线程。

上面示例的执行结果是不一定的，因为当解除阻塞后，CPU先执行那个线程是不定的。

CyclicBarrie在构建时支持传入一个Runnable方法，当阻塞的线程达到初始化值后，优先执行传入的Runnable方法。

CyclicBarrie可以通过其reset方法重置，因此CyclicBarrie方法可以复用。

### Semaphore

信号量Semaphore用来控制线程的数量。

##### 示例

``` java
Semaphore semaphore = new Semaphore(2);

IntStream.range(0, 4).forEach(i -> {
    new Thread(() -> {
        try {
            semaphore.acquire();
            System.out.println("线程开始执行");
            Thread.sleep(10000);
            System.out.println("线程结束执行");
            semaphore.release();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }).start();
});
```

``` text
线程开始执行
线程开始执行
线程结束执行
线程结束执行
线程开始执行
线程开始执行
线程结束执行
线程结束执行
```

##### 分析

Semaphore在初始化时会传入一个值，该值用来表示许可证的个数，其acquire方法表示获取一个许可证，其release方法表示释放一个许可证，获取不到许可证的线程会被阻塞。

许可证的数量可以通过Semaphore的方法修改。

### Exchanger

交换器Exchanger用来进行线程间数据交换。

##### 示例

``` java
Exchanger<String> exchanger = new Exchanger<>();

new Thread(() -> {
    try {
        System.out.println("子线程：" + exchanger.exchange("子线程数据"));
    } catch (Exception e) {
        e.printStackTrace();
    }
}).start();

System.out.println("主线程：" + exchanger.exchange("主线程数据"));
```

``` text
子线程：主线程数据
主线程：子线程数据
```