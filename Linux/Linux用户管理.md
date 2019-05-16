本文介绍了Linux用户管理相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 概述

Linux是一个多用户多任务的分时系统，任何想要使用Linux系统的用户都必须有一个合法的账户。

Linux通过用户管理更好的利用Linux系统的资源。

### 用户

##### UID

UID是Linux用来标识不同用户的数字。

##### 分类

* 普通用户

普通用户的UID一般大于1000，因为Linux普通用户的UID默认是从1000开始编号的，普通用户只能访问其自己的家目录，系统临时目录以及经过授权的的目录。

* 根用户

根用户也称为root用户，其UID为0，该账户拥有系统的完全控制权，其可以修改删除任何文件，运行任何命令。

* 系统用户

系统用户是指系统运行时所需的用户，其并不是真正的使用者，该用户不需要登陆，其是用来运行进程的，系统用户UID默认为201~999。

### 用户组

##### GID

GID是Linux用来标识不同用户组的数字。

##### 说明

在Linux中，用户组是用来划分管理用户的，每个用户都至少属于一个用户组。

### /etc/login.defs

该文件会存储用户的一些设定限制，例如GID和UID的值限定，该文件配置的优先级没有/etc/shadow文件配置的优先级高。

### 用户信息存储

Linux系统用来存储用户信息的重要两个文件是/etc/passwd和/etc/shadow。

Linux系统将基本信息存储在/etc/passwd中，将密码相关的信息存储在/etc/shadow中，密码会进行加密存储。

/etc/passwd文件每个用户都有权限，/etc/shadow文件只有root用户才有权限。

##### /etc/passwd

该文件每行都是用6个:分隔的7个字符串。

用户名:密码:UID:GID:说明:用户家目录:登录shell

密码会存储在/etc/shadow中，该文件中密码处统一用x。

##### /etc/shadow

该文件每行都是用8个:分隔的9个字符串。

用户名:加密后密码:密码的最近修改日:密码不可修改的天数:密码有效天数:密码失效前警告天数:密码失效宽限天数:账号失效日期:保留字段

密码最近修改日是从1970年1月1日至最近修改日期的天数。

密码不可修改的天数是指密码修改后不可再次修改的天数。

### 用户组信息存储

Linux系统用来存储用户组信息的文件是/etc/group。

##### /etc/group

该文件每行都是用3个:分隔的4个字符串。

用户组名:密码（实际没有使用，为x）:GID:组成员

### 账号管理

##### 用户

* 查看当前用户UID

``` shell
[root@0a8853f88de2 /]# id
uid=0(root) gid=0(root) groups=0(root)
```

* 查看当前用户所属用户组

``` shell
[root@0a8853f88de2 /]# groups
root
```

* 添加用户

``` shell
[root@0a8853f88de2 /]# useradd demo1
```

-u 指定UID

-g 指定用户组（是用户组名称不是GID）

-d 指定用户家目录

* 修改密码

``` shell
[root@0a8853f88de2 /]# passwd demo1
Changing password for user demo1.
New password:
BAD PASSWORD: The password is a palindrome
Retype new password:
passwd: all authentication tokens updated successfully.
```

普通用户只能修改自己的密码，passwd后不能加用户名。

* 修改用户

用户修改既可以通过usermod命令，也可以直接编辑/etc/passwd和/etc/shadow文件。

锁定用户/解锁用户（锁定密码，使密码验证不通过）。

``` shell
[root@0a8853f88de2 /]# usermod -L demo1
[root@0a8853f88de2 /]# usermod -U demo1
```

usermod根据不同的参数修改不同的信息。

* 删除用户

``` shell
[root@0a8853f88de2 /]# userdel demo1
```

默认只删除/etc/passwd和/etc/shadow文件中的记录。

-r 删除用户家目录和该用户的邮件。

##### 用户组

* 添加用户组

``` shell
[root@0a8853f88de2 /]# groupadd demo1
```

* 删除用户组

``` shell
[root@0a8853f88de2 /]# groupdel demo1
```

##### 检查用户信息

* 查看当前用户

``` shell
[root@ef3a95292bf4 /]# users
```

``` shell
[root@ef3a95292bf4 /]# who
```

``` shell
[root@ef3a95292bf4 /]# w
```

``` shell
[root@ef3a95292bf4 /]# finger
```

* 查看详细信息

``` shell
[root@ef3a95292bf4 /]# finger root
Login: root                             Name: root
Directory: /root                        Shell: /bin/bash
Never logged in.
No mail.
No Plan.
```

##### 切换用户

* 切换用户

``` shell
[root@ef3a95292bf4 /]# su demo1
[demo1@ef3a95292bf4 /]$ su
Password:
[root@ef3a95292bf4 /]# exit
exit
[demo1@ef3a95292bf4 /]$ exit
exit
[root@ef3a95292bf4 /]#
```

su默认切换到root用户。

exit命令表示退出当前用户并切换到前一个用户。

\- 不光切换用户，并将用户环境（例如用户家目录）一并切换。

* 以root用户操作

``` shell
[demo1@ef3a95292bf4 ~]$ sudo useradd demo2
```

配置/etc/sudoers，给demo1添加执行sudo命令的权限。

``` shell
[root@ef3a95292bf4 /]# visudo
```

``` text
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
demo1   ALL=(ALL)       ALL
```

sudoers可以是用来配置sudo命令的，例如配置sudo命令不需要密码。

``` text
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
demo1   ALL=(ALL)       NOPASSWD:ALL
```

### 添加用户流程

* 将相关数据存储在/etc/passwd，/etc/shadow文件中

* 创建家目录，家目录默认路径为/home/用户名

* 复制/etc/skel下所有文件至/home/用户名，/etc/skel为用户家目录模板

* 创建一个与该用户名一致的用户组，该用户组会将该用户添加进去

添加完用户是不能登录的，因为还没有设置密码，在/etc/shadow中，该用户加密后密码的位置为!!。