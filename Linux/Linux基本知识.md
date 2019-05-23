本文介绍了Linux基本知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 概述

Linux（Linux is not Unix）是一个类Unix的系统，其由芬兰人李纳斯 托瓦兹（Linux Torvalds）在赫尔辛基大学上学期间编写。

### Linux特点

* 免费开源

Linux现采用GPL协议。

* 模块化程度高

Linux内核设计分为进程管理，内存管理，进程间通信，虚拟文件系统，网络五部分，用户可以根据实际需要在内核中加入和移除模块。

* 广泛的硬件支持

* 安全稳定

* 多用户多任务

### Linux发行版

Linux发行版是Linux内核与软件管理的打包，不同发行版之间的区别体现在软件管理，其内核都来自Linux内核官网（[https://www.kernel.org](https://www.kernel.org)）。

常见的Linux发行版：RedHat，CentOS，Ubuntu，Debian，Fedora，Arch Linux，SUSE，OpenSUSE，SolusOS等。

### Linux主要分区

* 根分区（/）

* 交换（swap）分区

当内存不足时，会将交换分区当做内存使用，一般设置为物理内存的两倍。

* /boot分区

存放Linux系统启动所需文件。

### Grub

Grub（Grand Unified BootLoader）是系统引导工具，其用来加载内核，引导系统启动。

### DHCP

Dynamic Host Configuration Protocol，动态主机配置协议。用来给网络节点上的主机分配IP地址。

### X Server

Linux通过X Server低层程序来提供图形环境，用户不能直接与X Server交互，其需要通过图形环境与X Server进行交互。

### 终端

终端又叫tty，其是Teletype的简写。

图形环境下，Linux默认情况下提供6个虚拟终端，Ctrl+ALT+F1进入第一个终端，依次往下递增可以打开总共6个终端，Ctrl+ALT+F7返回桌面。

在正确的安装了图形环境的情况下，使用如下命令可以打开图形界面。

``` shell
startx
```

Shell中，#提示符表示当前用户是根用户，普通用户的提示符为$。

### Linux命令使用

Linux命令是严格大小写的。

##### date

日期

``` shell
[root@0a8853f88de2 /]# date
Tue May  7 14:39:11 UTC 2019
[root@0a8853f88de2 /]# date +%Y%m%d
20190507
```

##### ls

显示当前目录文件

``` shell
[root@0a8853f88de2 /]# ls
anaconda-post.log  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@0a8853f88de2 /]# ls -l
total 56
-rw-r--r--   1 root root 12082 Mar  5 17:36 anaconda-post.log
lrwxrwxrwx   1 root root     7 Mar  5 17:34 bin -> usr/bin
drwxr-xr-x   5 root root   360 May  7 14:37 dev
drwxr-xr-x   1 root root  4096 May  7 14:37 etc
drwxr-xr-x   2 root root  4096 Apr 11  2018 home
lrwxrwxrwx   1 root root     7 Mar  5 17:34 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Mar  5 17:34 lib64 -> usr/lib64
drwxr-xr-x   2 root root  4096 Apr 11  2018 media
drwxr-xr-x   2 root root  4096 Apr 11  2018 mnt
drwxr-xr-x   2 root root  4096 Apr 11  2018 opt
dr-xr-xr-x 133 root root     0 May  7 14:37 proc
dr-xr-x---   1 root root  4096 May  7 14:38 root
drwxr-xr-x  11 root root  4096 Mar  5 17:36 run
lrwxrwxrwx   1 root root     8 Mar  5 17:34 sbin -> usr/sbin
drwxr-xr-x   2 root root  4096 Apr 11  2018 srv
dr-xr-xr-x  13 root root     0 May  7 14:37 sys
drwxrwxrwt   7 root root  4096 Mar  5 17:36 tmp
drwxr-xr-x  13 root root  4096 Mar  5 17:34 usr
drwxr-xr-x  18 root root  4096 Mar  5 17:34 var
[root@0a8853f88de2 /]# ls -l anaconda-post.log
-rw-r--r-- 1 root root 12082 Mar  5 17:36 anaconda-post.log
```

##### cat

查看文件内容

``` shell
[root@0a8853f88de2 /]# cat anaconda-post.log
```

内容略

### 系统启动流程

* 加载BIOS

BIOS重要功能之一是检测自身硬件，如硬件没问题，则继续往下运行。

* 读取主引导记录

机器自检通过后，开始读取主引导记录（MBR，Main Boot Record），位于磁盘的第0柱面，第0磁道，第一个扇区。

MBR存放的是引导程序和分区信息，引导程序占用446字节，磁盘分区占用64字节，最后2字节是MBR的结束位。

该扇区空间是由专门的分区程序产生的。例如，fdisk.exe（Windows），fdisk命令（Linux）。

* 引导操作系统

RedHat和Centos默认使用Grub作为引导操作系统的程序。

由于Grub自身比较大，所以MBR一般写入的是Grub的地址，根据地址去读取Grub程序。

Grub程序会根据配置文件加载Kernel镜像，加载后运行init程序（内核加载后第一个运行的程序）。

* 系统初始化

init程序会根据初始化配置文件进行初始化工作，初始化工作中会根据配置运行相应的程序。

* 生成终端或启动图形环境等待登陆

### 系统运行级别

系统运行级别配置在初始化配置文件中，并根据配置读取不同的文件/etc/rcX.d（X代表0~6），不同的运行级别对应的服务也不一样。

|运行级别|说明|
|:----|:----|
|0|关机|
|1|单用户模式，没有网络连接，用来系统维护，例如修改root密码|
|2|多用户模式，没有网络连接|
|3|完全多用户模式，最常见的运行级|
|4|保留未使用|
|5|窗口模式，支持多用户，支持网络|
|6|重启|

### 使用帮助

* man page

``` shell
man ls
```

* info page

``` shell
info ls
```

* /usr/share/doc查看文档