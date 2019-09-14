### 泛型

泛型是java5中引入的特性，其本质是参数化类型，将类型当做参数使用。

##### 常见泛型标识符

|标识符|含义|
|:----|:----|
|T|type|
|E|element|
|K|key|
|V|value|

##### 类型擦除

泛型只在编译阶段有效，在运行前，其会将任何泛型相关的信息擦除，并在对象进入和离开时做类型检查及类型转化。

``` java
List<Integer> integerList = new ArrayList<>();
List<String> stringList = new ArrayList<>();
// 结果为true
System.out.println(integerList.getClass().equals(stringList.getClass()));
```

``` java
List stringListOne = new ArrayList();
stringListOne.add("Hello World");
// 需要转换类型
String stringOne = (String) stringListOne.get(0);

List<String> stringListTwo = new ArrayList<>();
stringListTwo.add("Hello World");
//无需转换类型
String stringTwo = stringListTwo.get(0);
```

``` java
List stringListOne = new ArrayList();
stringListOne.add("Hello World");
// 编译器不会报错
stringListOne.add(11);

List<String> stringListTwo = new ArrayList<>();
stringListTwo.add("Hello World");
// 编译器会报错
stringListTwo.add(11);
```

##### 基本类型

泛型不支持基本类型，使用时需使用其对应的包装类。

### 泛型类

``` java
public class Container<T> {

    private T t;

    public Container(T t) {
        this.t = t;
    }

    public T getT() {
        return t;
    }
}
```

``` java
Container<String> container = new Container<>("hello");
System.out.println(container.getT());
```

``` text
hello
```

泛型类也可以接受多个标识符，多个标识符用,号隔开，例如

``` java
public class Container<T, E> {

    private T t;
    private E e;

    public Container(T t, E e) {
        this.t = t;
        this.e = e;
    }

    public T getT() {
        return t;
    }

    public E getE() {
        return e;
    }
}
```

泛型类中的静态方法如用到泛型，需要使用泛型方法。

### 泛型接口

``` java
public interface Container<T> {

    String getClassName(T t);
}
```

``` java
public class ObjectContainer<T> implements Container<T> {

    private T t;

    public ObjectContainer(T t) {
        this.t = t;
    }

    @Override
    public String getClassName(T t) {
        return t.getClass().getName();
    }
}
```

``` java
public class StringContainer implements Container<String> {

    private String string;

    public StringContainer(String string) {
        this.string = string;
    }

    @Override
    public String getClassName(String string) {
        return string.getClass().getName();
    }
}
```


### 泛型方法

``` java
public <T> String getClassName(T t) {
    return t.getClass().getName();
}
```

``` java
Container container = new Container();
System.out.println(container.getClassName("hello"));
System.out.println(container.getClassName(0));
```

``` text
java.lang.String
java.lang.Integer
```

### 类型通配符<?>

在使用时，如果不确定具体的类型，我们可以使用通配符<?>来代指所有的类型。

泛型相当于是个模板，使用时需要指定具体的类型，而通配符?相当于一个对象，其是实实际际存在的，代表所有的类型。

``` java
public class ListDemo {

    public void getClassName(List<?> list) {
        System.out.println(list.getClass().getName());
    }
}
```

``` java
List<Integer> integerList = new ArrayList<>();
List<String> stringList = new ArrayList<>();

ListDemo listDemo = new ListDemo();
listDemo.getClassName(integerList);
listDemo.getClassName(stringList);
```

输出

``` text
java.util.ArrayList
java.util.ArrayList
```

因为通配符使用?来代指所有可能的泛型具体类型，所以我们现在不明确泛型具体的类型，无法做类型检验，于是我们不能使用类型相关的功能。

``` java
public void getClassName(List<?> list) {
    // 编译器报错
    list.add("hello");
    System.out.println(list.getClass().getName());
}
```

##### 类型通配符上限（上边界）

``` java
<? extends Number>
```

表示所传入的类型必须为Number的子类（可以为Number自身）。

##### 类型通配符下限（下边界）

``` java
<? super Integer>
```

表示所传入的类型必须为Integer的父类（可以为Integer自身）。

### 泛型数组

java不支持具体类型的泛型数组。因为泛型的类型擦除，运行阶段JVM无法确定数组中的类型是否相同。

``` java
// 报错
List<Integer>[] integerListArr = new ArrayList<Integer>[5];
```

通配符可以类型的数组支持。

``` java
List<?>[] integerListArr = new ArrayList<?>[5];

List<Integer> integerList = new ArrayList<>();
integerList.add(0);
integerList.add(1);
integerListArr[0] = integerList;

List<String> stringList = new ArrayList<>();
stringList.add("Hello");
stringList.add("World");
integerListArr[1] = stringList;

System.out.println(integerListArr[0].toString());
System.out.println(integerListArr[1].toString());
```

``` text
[0, 1]
[Hello, World]
```