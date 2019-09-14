### 概述

Shell是一个命令解释器，其提供了与内核交互的通道。

Shell本身还是一个解释性语言，其可以用来编写Shell脚本。

### Shell版本

Linux Shell版本比较多，以下为比较知名的Shell版本。

|名称|位置|
|:----|:----|
|Bourne Shell|/bin/sh或/usr/bin/sh|
|C Shell|/usr/bin/csh|
|Korn Shell|/usr/bin/ksh|
|Bourne Again Shell|/bin/bash|

目前最常用的就是Bourne Again Shell，简称Bash，其完全兼容Bourne Shell。

### 工作模式

* 互动模式

直接在Shell中使用命令。

* 脚本模式

编写Shell脚本，运行Shell脚本。

### 编写Shell脚本

``` shell
#!/bin/bash

echo "hello world!"
```

``` text
#!/bin/bash用来声明执行该脚本的解释器。
```

``` shell
chmod +x hello.sh
```

授予该脚本执行权限。

### Shell脚本运行方式

* 可执行程序

``` shell
./hello.sh
```

需要加./告知系统是执行当前路径下的hello.sh脚本，如果不加系统会去PATH路径中寻找。

如果该脚本在PATH中，例如（/bin，/sbin，/usr/bin，/usr/sbin），是可以直接运行的，不需要加./。

``` shell
hello.sh
```

* 作为解释器参数运行

``` shell
/bin/bash hello.sh
```

+x参数可以观察脚本的运行情况以方便查询问题。

### Shell内建命令

Shell内建命令是指Shell自身提供的命令。

通常来说，内建命令的效率要比外部命令的效率高。

##### 常见Shell内建命令

* type

判断命令类型。

``` shell
[root@8091b99b9eb0 /]# type pwd
pwd is a shell builtin
[root@8091b99b9eb0 /]# type ls
ls is aliased to `ls --color=auto'
[root@8091b99b9eb0 /]# type mkdir
mkdir is /usr/bin/mkdir
```

* .

执行程序，在下面的示例中，即使hello.sh没有授予执行权限其也可以执行。

``` shell
[root@8091b99b9eb0 /]# . ./hello.sh
hello world!
```

* alias

别名。

``` shell
[root@8091b99b9eb0 /]# alias
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
```

自定义别名

``` shell
[root@8091b99b9eb0 /]# alias demo='ll'
```

删除自定义别名

``` shell
[root@8091b99b9eb0 /]# unalias demo
```

上述别名只是在当前shell中生效，要想永久生效，需配置在用户家目录中的.bashrc文件。

* cd

切换目录。

* echo

打印字符。

``` shell
[root@8091b99b9eb0 /]# echo "hello"
hello
```

* exec

执行命令取代当前shell。

``` shell
[root@8091b99b9eb0 /]# exec /bin/sh
sh-4.2# exec echo "hello"
hello
```

exec echo "hello"是将当前进程替换为执行echo "hello"命令进程，执行完echo "hello"命令终止。

* exit

退出Shell。

* kill

终止进程。

* pwd

显示当前目录。

* test

测试表达式。