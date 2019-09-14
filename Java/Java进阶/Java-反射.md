### 概述

反射是Java重要的一个功能，其可以在运行期间从类对象中获取构造器，属性，方法等信息并执行相应操作。

### Class对象

字节码文件加载到JVM中，其会将字节码文件中的相关数据存储在方法区（Java 8使用元空间替代了方法区），并在堆中创建Class对象来封装方法区中相应的数据，Class对象的类为java.lang.Class类，其在java.lang包下，文件存在于rt.jar包中。

##### Class

Class类是Class对象的类。

Class类的构造函数是私有的，其不会被使用，Class对象只能通过JVM生成。

### java.lang.reflect包

该包提供了反射的相关功能。

|名称|说明|
|:----|:----|
|Member|表示单个成员（属性或方法）或构造器的标识信息|
|Constructor|表示构造函数的相关信息|
|Field|表示域的相关信息|
|Method|表示方法的相关信息|

### 相关方法

##### 获取Class对象

* Class.forName

* 根据包路径和名称获取

* 使用Object的getClass方法

##### 判断对象所属类

* instanceof

* Class对象的isInstance（Native方法）

##### 创建实例

* Class对象的newInstance方法

* 使用Constructor对象的newInstance方法

### 示例

Class类对象获取Constructor,Field,Method方法默认是获取public修饰的，带Declared的方法是获取所有的。

``` java
Class peopleClassObjectOne = Class.forName("People");
Class peopleClassObjectTwo = People.class;
People peopleObjectOne = new People("小白", 18);
Class peopleClassObjectThree = peopleObjectOne.getClass();

System.out.println(peopleClassObjectOne == peopleClassObjectTwo);
System.out.println(peopleClassObjectOne == peopleClassObjectThree);
System.out.println(peopleObjectOne instanceof People);
System.out.println(peopleClassObjectOne.isInstance(peopleObjectOne));

People peopleObjectTwo = (People) peopleClassObjectOne.newInstance();
Constructor constructor = peopleClassObjectOne.getConstructor(String.class, int.class);
People peopleObjectThree = (People) constructor.newInstance("小黑", 18);

Field[] publicFields = peopleClassObjectOne.getFields();
for (Field publicField : publicFields) {
    System.out.println(publicField.getName());
}

Field publicField = peopleClassObjectOne.getField("age");
System.out.println(publicField.getType());

Field[] declaredFields = peopleClassObjectOne.getDeclaredFields();

for (Field declaredField : declaredFields) {
    System.out.println(declaredField.getName());
}

Method method = peopleClassObjectOne.getMethod("sayName");
method.invoke(peopleObjectOne);
```

``` text
true
true
true
true
age
int
name
age
我的名字是小白
```

### 反射的缺点

* 性能开销大

反射是在运行期间而不是编译期间，所以其无法用到编译优化。

* 安全限制

反射只能在没有安全限制的环境中运行，例如Applet等需要在安全限制环境中运行的应用则无法使用。

* 破坏封装性

反射破坏了封装性，其可以执行一些正常情况下无法执行的操作，这样有可能带来一些不可预知的影响。