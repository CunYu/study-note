### JVM配置格式

* 标准参数（-）

所有JVM实现都支持，并且向后兼容。

* 非标准参数（-X）

默认JVM支持，但不保证所有JVM实现都支持，也不保证向后兼容。

* 非稳定参数（-XX）

在不同的JVM实现中的会有不同，而且功能也不稳定，将来有可能会随时被取消。

参数分为boolean类型和非boolean类型。

### 常见参数

* -

``` text
-version

-help

-server

-client

// 32位
-d32

// 64位
-d64

// 设置系统属性
-D

// 目录和zip/jar文件的类搜索路径，用:分隔
-cp
-classpath
```

* -X

``` text
// 设置初始堆大小，默认为物理内存的1/64（不超过1G），当堆空闲内存达到70%（比例可配置-XX:MaxHeapFreeRation）时，堆内存会减少至-Xms指定的内存
-Xms

// 设置堆最大值，默认为物理内存的1/4（不超过1G），当堆空闲内存不足40%（比例可配置-XX:MinHeapFreeRation）时，堆内存会增加至-Xmx指定的内存
-Xmx
// 建议-Xms和-Xmx设置为一样的值

// 设置青年代大小，青年代会影响老年代，建议青年代大小为堆的3/8
-Xmn

// 设置线程堆栈大小
-Xss
```

* -XX

``` text
// 设置青年代大小
-XX:NewSize

// 设置青年代最大值
-XX:MaxNewSize

// 设置元空间大小
-XX:MetaspaceSize

// 设置元空间最大值
-XX:MaxMetaspaceSize

// 设置Eden和Survivor的比例，默认为8
-XX:SurvivorRation

// 直接晋升老年代的对象大小
-XX:PretenureSizeThreshold

// 晋升老年代的年龄
-XX:MaxTenuringThreshold

// 并行GC的线程数
-XX:ParallelGCThreads

// 是否允许担保失败
-XX:HandlePromotionFailure

// JVM Client模式的默认值，使用Serial + Serial Old收集器
-XX:+UseSerialGC

// 使用ParNew + Serial Old收集器
-XX:+UseParNewGC

// 使用ParNew + CMS +Serial Old收集器
-XX:+UseConcMarkSweepGC

// JVM Server模式的默认值，使用Parallel Scavenge + Serial Old收集器
-XX:+UseParallelGC

// 使用Parallel Scavenge + Parallel Old收集器
-XX:+UseParallelOldGC
```