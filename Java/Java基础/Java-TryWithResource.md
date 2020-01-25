### 概述

Java7为了简化资源的关闭代码，使其变得更加优雅，其提供了TryWithResource功能。

### 对比

* 传统

``` java
InputStream inputStream = null;
try {
    inputStream = new FileInputStream(new File("/home/cunyu/Documents/demo.txt"));
    int i;
    while ((i = inputStream.read()) != -1) {
        System.out.print((char) i);
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    if (null != inputStream) {
        try {
            inputStream.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

``` text
Hello World!
```

* TryWithResource

``` java
try (InputStream inputStream = new FileInputStream(new File("/home/cunyu/Documents/demo.txt"))) {
    int i;
    while ((i = inputStream.read()) != -1) {
        System.out.print((char) i);
    }
} catch (Exception e) {
    e.printStackTrace();
}
```

``` text
Hello World!
```

### 说明

TryWithResource实际上是对传统写法的封装，其编译后和传统写法编译后的语法是一样的，但是其只适用于实现AutoCloseable接口的资源。

``` java
try {
    InputStream inputStream = new FileInputStream(new File("/home/cunyu/Documents/demo.txt"));
    Throwable var2 = null;

    try {
        int i;
        try {
            while((i = inputStream.read()) != -1) {
                System.out.print((char)i);
            }
        } catch (Throwable var12) {
            var2 = var12;
            throw var12;
        }
    } finally {
        if (inputStream != null) {
            if (var2 != null) {
                try {
                    inputStream.close();
                } catch (Throwable var11) {
                    var2.addSuppressed(var11);
                }
            } else {
                inputStream.close();
            }
        }

    }
} catch (Exception var14) {
    var14.printStackTrace();
}
```