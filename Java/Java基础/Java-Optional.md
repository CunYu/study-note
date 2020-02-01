### 概述

Java 8中新引入了Optional包装类来使代码变得更优雅。

### 示例

* 创建

``` java
// 创建一个optional，optional包装的值为null
Optional<String> optionalOne = Optional.empty();
// 创建一个optional，optional包装的值不能为null，为null则抛出java.lang.NullPointerException异常
Optional<String> optionalTwo = Optional.of("Hello World");
// 创建一个optional，optional包装的值可以为null
Optional<String> optionalThree = Optional.ofNullable(null);
```

* 使用

``` java
Optional<String> optional = Optional.ofNullable("Hello World!");

// 如果optional包装值为null则抛出异常
System.out.println(optional.get());

System.out.println(optional.isPresent());
optional.ifPresent(str -> System.out.println(str));

// 如果optional包装值为null则返回设置的字符串
System.out.println(optional.orElse("null"));
System.out.println(optional.orElseGet(() -> "null"));
// 如果optional包装值为null则返回设置的异常
System.out.println(optional.orElseThrow(RuntimeException::new));

// 如果option包装值不为null，则对option包装值进行操作并返回新的option对象，如果option包装值为null，则返回包装值为null的option
optional = optional.map(String::toUpperCase);
System.out.println(optional.get());
optional = optional.flatMap(str -> Optional.of(str.toUpperCase()));
System.out.println(optional.get());

optional = optional.filter(str -> str.startsWith("HELLO"));
System.out.println(optional.get());
optional = optional.filter(str -> str.startsWith("test"));
System.out.println(optional.isPresent());
```

``` text
Hello World!
true
Hello World!
Hello World!
Hello World!
Hello World!
HELLO WORLD!
HELLO WORLD!
HELLO WORLD!
false
```