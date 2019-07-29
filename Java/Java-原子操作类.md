本文介绍了Java原子操作类相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 概述

Java原子操作类是从JDK 1.5开始提供的，其位于java.util.concurrent.atomic包下，其提供了一种简单，高效，线程安全的更新变量的方式，其操作是原子性的。

### 常见原子操作类

##### 原子操作基本类型类

|名称|说明|
|:----:|:----:|
|AtomicBoolean|原子操作Boolean类|
|AtomicInteger|原子操作Integer类|
|AtomicLong|原子操作Long类|

##### 原子操作数组类

|名称|说明|
|:----:|:----:|
|AtomicIntegerArray|原子操作Integer数组类|
|AtomicLongArray|原子操作Long数组类|
|AtomicReferenceArray|原子操作引用数组类|

##### 原子操作引用类

|名称|说明|
|:----:|:----:|
|AtomicReference|原子操作引用类|
|AtomicReferenceFieldUpdater|原子操作引用中字段类|
|AtomicMarkableReference|原子操作带有标记的引用类|

##### 原子操作字段类

|名称|说明|
|:----:|:----:|
|AtomicIntegerFieldUpdater|原子操作Integer字段类|
|AtomicLongFieldUpdater|原子操作Long字段类|
|AtomicStampedReference|原子操作带有版本号的引用类|

### 示例

##### AtomicInteger

``` java
AtomicInteger atomicInteger = new AtomicInteger(0);
for (int i = 0; i < 5000; i++) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            //通过for循环调用CAS方法实现
            atomicInteger.getAndIncrement();
        }
    }).start();
}
Thread.sleep(5000);
System.out.println(atomicInteger);
```

输出

``` text
5000
```

##### AtomicIntegerArray

``` java
int[] intArray = {0, 0};
AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(intArray);
for (int i = 0; i < 5000; i++) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            atomicIntegerArray.addAndGet(0, 1);
            atomicIntegerArray.addAndGet(1, 2);
        }
    }).start();
}
Thread.sleep(5000);
Arrays.stream(intArray).forEach(System.out::println);
System.out.println(atomicIntegerArray.toString());
```

输出

``` text
0
0
[5000, 10000]
```

##### AtomicReference

``` java
@Data
public class People {

    private String name;
    private int age;

    public People(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

``` java
People people = new People("xiaoBai", 10);
AtomicReference<People> peopleAtomicReference = new AtomicReference<>();
peopleAtomicReference.set(people);
People peopleUpdate = new People("xiaoBai", 15);
peopleAtomicReference.compareAndSet(people, peopleUpdate);
System.out.println(people.toString());
System.out.println(peopleAtomicReference.toString());
```

输出

``` text
People(name=xiaoBai, age=10)
People(name=xiaoBai, age=15)
```

##### AtomicIntegerFieldUpdater

``` java
@Data
public class People {

    private String name;
    // AtomicIntegerFieldUpdater更新的字段必须用public volatile声明
    public volatile int age;

    public People(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

``` java
People people = new People("xiaoBai", 10);
AtomicIntegerFieldUpdater<People> atomicIntegerFieldUpdater = AtomicIntegerFieldUpdater.newUpdater(People.class, "age");
people.setAge(atomicIntegerFieldUpdater.getAndIncrement(people));
System.out.println(people.toString());
```

输出

``` text
People(name=xiaoBai, age=10)
```