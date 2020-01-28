### 概述

Java 8中引入了Lambda来表示函数式接口，其使代码变得更加优雅。

### 格式

``` text
(参数, 参数, ...) -> {表达式}
```

如果参数只有一个，()可以省略。

如果表达式只有一行，{}可以省略。

如果参数只用来传递，那么其可以用::符号来简化。

``` java
IntStream.range(0, 5).forEach(i -> {
    System.out.println(i);
});

IntStream.range(0, 5).forEach(System.out::println);
```

### 函数式接口

函数式接口是指接口中只有一个方法（不包含默认方法，接口可以有多个默认方法）的接口，函数式接口最好用@FunctionalInterface修饰，这样就不会误给其添加多余的方法导致lambda表达式报错。

lambda表达式是用来表示函数式接口的。

### function包常用函数式接口

* Predicate

``` java
public static void main(String[] args) {

    List<String> list = Arrays.asList("one", "two", "three", "four", "five");
    List<String> filteredData = filterData(list, str -> !str.endsWith("o"));
    filteredData.forEach(System.out::println);
}

public static List<String> filterData(List<String> list, Predicate<String> predicate) {
    List<String> filteredData = new ArrayList<>();
    list.forEach(str -> {
        if (predicate.test(str)) {
            filteredData.add(str);
        }
    });
    return filteredData;
}
```

``` text
one
three
four
five
```

* Consumer

``` java
public static void main(String[] args) {
    List<String> list = Arrays.asList("one", "two", "three", "four", "five");
    dealData(list, System.out::println);
}

public static void dealData(List<String> list, Consumer<String> consumer) {
    list.forEach(str -> consumer.accept(str));
}
```

``` text
one
two
three
four
five
```

* Function

``` java
public static void main(String[] args) {

    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    List<String> filteredData = convertData(list, String::valueOf);
    filteredData.forEach(System.out::println);
}

public static List<String> convertData(List<Integer> list, Function<Integer, String> function) {

    List<String> convertedData = new ArrayList<>();
    list.forEach(integer -> {
        convertedData.add(function.apply(integer));
    });
    return convertedData;
}
```

``` text
1
2
3
4
5
```