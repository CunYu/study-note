### 包装类

在java中，有8种基本类型，然而基本类型并不是对象，所以使用起来会有很多的不便，因此java提供包装类（Wrapper Class）来对基本类型进行包装。

|基本类型|包装类|字节数|
|:----|:----|:----|
|short|Short|2|
|int|Integer|4|
|long|Long|8|
|float|Float|4|
|double|Double|8|
|byte|Byte|1|
|char|Character|2|
|boolean|Boolean|1|

### 装箱拆箱

##### 定义

###### 装箱

基本类型转换为包装类型。

###### 拆箱

包装类型转换为基本类型。

##### 示例

``` java
Integer i = new Integer(1);
int j = i.intValue();
```

java5开始支持基本类型和包装类型的自动转化，即自动装箱，自动拆箱（Autoboxing and unboxing）。

``` java
Integer i = 1;
int j = i;
```

### 自动装箱拆箱原理

以上面自动装箱拆箱示例为例，自动装箱本质是调用了Integer.valueOf(1)方法。自动拆箱本质是调用了Integer.intValue()方法。

##### 自动装箱

Integer.valueOf方法相关源码

``` java
public static Integer valueOf(int i) {
    if (i >= Integer.IntegerCache.low && i <= Integer.IntegerCache.high)
        return Integer.IntegerCache.cache[i + (-Integer.IntegerCache.low)];
    return new Integer(i);
}
```

Integer中的内部静态类IntegerCache相关源码

``` java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert Integer.IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

从源码中可知，Integer中定义了一个IntegerCache静态内部类，该内部类加载时会初始化一个Integer数组，该数组会存放low到high的int值，low为-128，high默认为127，其也可以通过JVM配置中的-XX:AutoBoxCacheMax=指定，自动装箱过程中，如果值在low和high之间（包括low和high），则返回数组中相对应的值。

|包装类|默认缓存|上限值是否可以设置|
|:----|:----|:----|
|Short|[-128, 127]|N|
|Integer|[-128, 127]|Y|
|Long|[-128, 127]|N|
|Float|-|-|
|Double|-|-|
|Byte|[-128, 127]|N|
|Character|[0, 127]|N|
|Boolean|false, true|N|

##### 自动拆箱

intValue方法相关源码

``` java
public int intValue() {
    return value;
}
```

直接返回值。

### 装箱拆箱场景

* 类型装换

例如

``` java
Integer i = 1;
int j = i;
```

* 运算

进行+，-，*，/等运算时，会拆箱。

例如

``` java
Integer i = 1;
Integer j = 1;
i = i + j;
```

进行运算时，i，j会先拆箱，运算完后结果再装箱。

* ==

如果两边都是基本类型则比较值。

如果两边都是包装器类型则比较地址。

如果一边是基本类型，一边是包装类型，会对包装类型进行拆箱，然后比较值。

### 备注

本文所涉及Java源码对应Java版本为1.8.0_162。