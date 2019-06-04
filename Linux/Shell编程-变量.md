本文介绍了Shell编程-变量相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 概述

变量是指变化的量，其本身是一个用来存储数据的存储空间。

### Shell变量特点

* Shell变量是一种弱类型的变量。

* Shell变量分为局部变量和环境变量。

* Shell变量时一种弱类型的变量，其使用时不需要指定具体的类型，也不需要遵循先声明后使用的规定。

### Shell变量类型

##### 局部变量

局部变量只在声明其的Shell中有效，每个Shell都有自己的命名空间，互不干扰。

local可以显示的声明局部变量，但是只能在函数内声明，只在函数内生效，其只存在于局部的命名空间内，不影响全局变量。

##### 环境变量

环境变量又称全局变量，其在声明其的Shell及其衍生的任何子Shell或进程中有效。

环境变量可以传递到其子Shell中，但子Shell的环境变量父Shell不能获取。

export声明的变量为环境变量。

``` shell
export demo=123
```

##### Shell变量

Shell自身预设的变量。

例如

``` shell
[root@8091b99b9eb0 /]# echo $BASH
/bin/bash
[root@8091b99b9eb0 /]# echo $BASH_VERSION
4.2.46(2)-release
[root@8091b99b9eb0 /]# echo $PWD
/
[root@8091b99b9eb0 /]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

### 变量命名规则

* 只能使用英文字母，数字和下划线，首字母不能为数字。

* 不能使用bash的关键字。

### 变量操作

##### 基本操作

``` shell
#!/bin/bash

:<<EOF
赋值
变量赋值=号两边不能有空格
变量值有空格的情况下必须使用引号（单引号和双引号都可以使用）
单引号里的任何字符都会原样输出（例如$只是普通的一个符号）
单引号不能使用转义符
双引号里面可以使用变量
双引号可以使用转义符
EOF
demo_1=hello_1
demo_2="hello_2"

# :<<多行注释可以用其他部分符号替代EOF
:<<!
只读变量
只读变量不能改动和删除
!
demo_3="hello_3"
readonly demo_3

# 拼接字符串
demo_4=${demo_1:0:5}" world!"

# 取值
echo $demo_1
echo ${demo_2}
echo ${demo_3}
echo ${demo_4}

# 判断字符串长度
echo ${#demo_4}

# 截取字符串
echo ${demo_4:0:5}

# 删除变量
unset demo_1

echo ${demo_1}
```

``` text
hello_1
hello_2
hello_3
hello world!
12
hello

```

##### 位置参数

``` shell
#!/bin/bash

:<<EOF
$后面只有一位的时候，可以不用{}
${0}表示脚本本身，${1}表示第一个参数，后面依次
$#表示参数的个数
$@和$*表示所有的参数
EOF
echo "name: ${0}"
echo "param: ${1}"
echo "param: ${2}"
echo "num: ${#}"
echo "$@"
echo "$*"
```

``` shell
[root@8091b99b9eb0 home]# ./position.sh param1 param2
name: ./position.sh
param: param1
param: param2
num: 2
param1 param2
param1 param2
```

##### 脚本/命令返回值

在Linux系统中，正常执行的命令和脚本应该以0位其返回值，非0表示命令或脚本执行出了问题。

$?可以获取上一个命令或脚本执行后的返回值

``` shell
[root@8091b99b9eb0 home]# pwd
/home
[root@8091b99b9eb0 home]# echo $?
0
```

##### 数组

Shell只支持一维数组。

``` shell
#!/bin/bash

# Shell中，数组初始化后还可以改变大小
array_1=("1" "2" "3")
array_2=()
array_2[0]=4
array_2[1]=5

# 读取数组
echo ${array_1[0]}

# 获取数组内所有元素
echo ${array_1[@]}
echo ${array_2[*]}

# 数组长度
echo ${#array_1[@]}
echo ${#array_2[*]}

# 截取数组
echo ${array_1:0:1}

# 删除数组
unset array_1[0]
unset array_2

echo ${array_1[@]}
echo ${array_2[*]}
```

``` text
1
1 2 3
4 5
3
2
1
2 3

```