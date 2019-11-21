### 概述

句柄是资源的一个标识符，其存在于内存中，其指向资源在内存中的区域。

### Linux限制

Linux中限制了一个进程可以持有的最大句柄数，默认值为1024。

* 查看进程可持有最大句柄数

``` shell
ulimit -n
```

* 临时设置进程可持有最大句柄数

``` shell
ulimit -n 2048
```

* 永久设置进程可持有最大句柄数

配置文件/etc/security/limits.conf。

``` shell
root soft nofile 1000000
root hard nofile 1000000
```

root表示root用户。

soft nofile指的是软性极限，超过这个值会发出警告，hard nofile指的是硬性极限，这是最大值，不能超过这个值。

* 查看系统最大句柄数

``` shell
cat /proc/sys/fs/file-max
```

* 修改系统最大句柄数

``` shell
echo fs.file-max = 10000000 >> /etc/sysctl.conf
```

* 查看进程句柄数

``` shell
cd /proc/xxx/fd
```

xxx为进程ID号，该目录内容为该进程持有的句柄。

``` shell
ls | wc -l
```

该命令可以统计该目录文件数。