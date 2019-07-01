本文介绍了Java字符串的相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 概述

字符串是程序中常见的数据类型，字符串是不可变的，String类用final修饰，其不可以被继承。

Java 8中，String通过字符数组（char[]）来存储。

Java 9中，String通过字节数组(byte[])来存储，并用coder（byte）来标识所使用的编码。

### String不可变

String的相关方法会返回一个新的String对象，其并不会对原有对象进行修改。

* String大多数情况下是用来传递表示信息的，所以其不能被修改比较好，保证了数据的安全。

* String Pool的需要。

* 缓存hash值，String类中有hash（int）属性，由于String不可变的原因，其可以将hash值存储下来，避免hash重复计算。

### +操作符

+是Java中的一个操作符，其可以用来连接String，+和+=是Java中仅有的两个重载操作符，Java不允许程序员重载操作符。

+操作符在编译时会引入StringBuilder，其连接String的功能是通过StringBuilder的append方法实现的，其会先初始化一个空的StringBuilder，然后通过append连接各个字符串，最后通过toString方法返回一个String。

``` java
String s1 = "Hello";
String s2 = " ";
String s3 = "World!";
// 编译器会通过引入StringBuilder对其进行优化
String s4 = s1 + s2 + s3;
// 下列情况，编译器不会引入StringBuilder对其进行优化
// 编译器编译的时候就会把其编译为String s5 = "Hello World!"
String s5 = "Hello" + " " + "World!";
```

虽然编译器会自动优化+操作符，但是涉及到String的连接时，还是建议不要使用+操作符，尤其在有循环的时候。

### String Pool

字符串在编程中大量使用，而字符串本身又不可以修改，所以可以使用字符串常量池来存储字符串，进而节省内存。

``` java
/**
 * s1和s2字符串的创建过程是一样的
 * 其会去字符串常量池中查找是否已有该字符串（通过equal方法比较）
 * 如果字符串常量池存在，那么返回字符串常量池中该字符串引用
 * 如果字符串常量池中不存在，那么将字符串添加到字符串常量池中，并返回字符串常量池中该字符串引用
 */
String s1 = "Hello World!";
String s2 = s1.intern();
```

在Java7之前，字符串常量池保存在方法区中，在Java7中，字符串常量池保存在堆中。

### StringBuilder和StringBuffer

StringBuilider和StringBuffer是可变的字符序列，也就是可修改的字符串。

StringBuilider和StringBuffer提供了很多修改自身的方法。

StringBuilder是线程不安全的，StringBuffer是线程安全的，其使用synchronized修饰自身方法。