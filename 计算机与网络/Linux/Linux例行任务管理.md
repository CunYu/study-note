### 一次性任务（at）

at命令依赖于atd服务。

* atd服务安装

``` shell
[root@8091b99b9eb0 /]# yum install at
```

* 启动atd服务

``` shell
[root@8091b99b9eb0 /]# systemctl start atd
```

* 开机启动atd服务

``` shell
[root@8091b99b9eb0 /]# systemctl enable atd
```

* 设置一次性任务

``` shell
[root@8091b99b9eb0 /]# at now + 5 minutes
at> touch hello.txt
at> <EOT>
job 9 at Mon May 20 14:36:00 2019
```

``` text
<EOT>是Ctrl+D快捷键
```

* 查看任务

``` shell
[root@8091b99b9eb0 /]# atq
9       Mon May 20 14:36:00 2019 a root
```

* 删除任务

``` shell
[root@8091b99b9eb0 /]# atrm 9
```

##### 相关文件说明

at任务会保存在/var/spool/at路径下。

如果存在/etc/at.allow文件，则只有该文件内的用户才能使用at。

如果不存在/etc/at.allow文件，但存在/etc/at.deny文件，则只有不在该文件内的用户才能使用at。

如果/etc/at.allow文件和/etc/at.deny文件都不存在，则只有root用户才能使用at。

### 周期性任务（cron）

cron命令依赖于crond服务。

* cron服务安装

``` shell
[root@8091b99b9eb0 /]# yum install crontabs
```

* 启动服务

``` shell
[root@8091b99b9eb0 /]# systemctl start crond
```

* 开机启动atd服务

``` shell
[root@8091b99b9eb0 /]# systemctl enable crond
```

* 创建任务

``` shell
[root@8091b99b9eb0 /]# crontab -e
```

``` text
* * * * * systemctl restart crond

分 时 日 月 周

* 表示每个单位都执行

/ 表示每过多少个单位执行

- 表示单位范围

, 多个单位值之间分隔符
```

* 查看任务

``` shell
[root@8091b99b9eb0 /]# crontab -l
* * * * * systemctl restart crond
```

crontab -u + 用户名 查看指定用户的任务

* 删除所有任务

``` shell
[root@8091b99b9eb0 /]# crontab -r
```

##### 相关文件说明

cron任务会保存在/var/spool/cron路径下。

如果存在/etc/cron.allow文件，则只有该文件内的用户才能使用cron。

如果不存在/etc/cron.allow文件，但存在/etc/cron.deny文件，则只有不在该文件内的用户才能使用cron。

如果/etc/cron.allow文件和/etc/cron.deny文件都不存在，则只有root用户才能使用cron。