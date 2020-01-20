### jps

``` shell
jps
```

输出正在运行的Java进程。

* -l

输出主类全路径。

* -m

输出main函数参数。

* -v

输出JVM参数。

* -q

只输出进程号。

### jinfo

``` shell
jinfo xxx
```

xxx表示进程号，输出JVM运行时的相关信息。

### jstat

``` shell
jstat xxx
```

xxx表示进程号，输出JVM运行时的状态数据。

* -gc

输出gc相关数据。

* -class

输出class相关数据。

### jmap

``` shell
jmap xxx
```

xxx表示进程号，输出JVM运行时的内存信息。

* -heap

输出堆信息。

* -dump:live,format=b,file=name.hprof

name表示Heap Dump文件名称，xxx表示进程号，输出Heap Dump文件，生成Heap Dump时，JVM会暂停所有线程。

Heap Dump可以通过JVM参数配置在某些情况下生成，例如在发生OOM错误时，Full GC前或Full GC后。

### jhat

``` shell
jhat xxx
```

xxx表示文件名。

该命令可以分析Heap Dump文件。

### jstack

``` shell
jstack xxx
```

xxx表示进程号，输出线程快照信息。

* -F

强制输出线程Dump信息。

* -m

输出Native信息。

* -l

输出锁附加信息。