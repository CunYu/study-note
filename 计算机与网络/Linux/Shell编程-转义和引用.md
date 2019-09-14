### 概述

Shell中有两类字符，一种是普通纯文本，没有任何含义，另一种是元字符，Shell的保留字符，其在Shell有特殊的含义。

为了消除特殊字符的功能，我们需要用到转义和引用。

### 转义

转义是指使用转义符引用单个字符，从而去除其特殊的含义。

##### 转义符\

转义符\可以使单个带有特殊含义的字符失去其特殊的含义。

``` shell
#!/bin/bash

echo $hello

# \符号放在需要的转移的单个字符前
echo \$hello
```

``` text

$hello
```

### 引用

引用是将字符串用特定符号包括起来，使其包含特殊含义的字符失去其特殊的含义。

##### 分类

* 双引号

* 单引号

* 反引号

* 转义符

##### 双引号引用

双引号引用是指使用双引用包括的引用，其称为部分引用或弱引用。双引号引用中$，`，\仍然有其特殊含义。

双引号中可以包含单引用，单引号并没有特殊含义。

``` shell
#!/bin/bash

hello=hello
echo $hello
echo "$hello"

demo="a    b    c"
echo $demo
echo "$demo"
```

``` text
hello
hello
a b c
a    b    c
```

##### 单引号引用

单引号引用是指使用单引号包括的引用，其称为全引用或强引用。

``` shell
#!/bin/bash

hello=hello
echo '$hello'
```

``` text
$hello
```

##### 反引号引用

反引号引用是指用\`号包括的引用，`号包括的值会解释为系统命令。

``` shell
#!/bin/bash

date_1=`date +"%Y%m%d"`
echo ${date_1}

:<<EOF
$()只在Bash Shell中有效
$()支持嵌套
EOF
date_2=$(date +"%Y%m%d")
echo ${date_2}

# 即使命令输出结果是多行的，变量获取到值也是一行的
info=`finger root`
echo ${info}
```

``` text
20190602
20190602
Login: root Name: root Directory: /root Shell: /bin/bash Last login Thu May 30 10:37 (UTC) on pts/2 No mail. No Plan.
```