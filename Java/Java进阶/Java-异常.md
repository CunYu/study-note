### 概述

Java使用异常来表示程序不正常的情况。

### 异常体系

Throwable是异常类的根类，其有两个重要的子类，Error和Exception。

Error是指非用户程序的异常，Exception是指用户程序的异常。

Exception子类中，除了运行时异常外都是检查性异常，检查性异常在编译时就会发现并需要做相应的处理。

Error子类都是非检查性异常。

### 异常处理机制

``` java
public static void main(String[] args) {

    test();
}

public static void test() {

    try {
        System.out.println("Hello");
    } catch (Exception e) {
        System.out.println("Catch Exception");
    } finally {
        System.out.println("Finally");
    }
}
```

``` text
Hello
Finally
```

``` java
public static void main(String[] args) {

    test();
}

public static void test() {

    try {
        System.out.println("Hello");
        throw new RuntimeException("Exception");
    } catch (Exception e) {
        System.out.println("Catch Exception");
    } finally {
        System.out.println("Finally");
    }
}
```

``` text
Hello
Catch Exception
Finally
```

``` java
public static void main(String[] args) {

    try {
        test();
    } catch (Exception e) {
        System.out.println("Main Catch Exception");
    }
}

public static void test() {

    try {
        System.out.println("Hello");
        throw new RuntimeException("Exception");
    } catch (Exception e) {
        System.out.println("Catch Exception");
        throw new RuntimeException("Exception");
    } finally {
        System.out.println("Finally");
    }
}
```

``` text
Hello
Catch Exception
Finally
Main Catch Exception
```