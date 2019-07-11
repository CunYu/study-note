本文介绍了java类加载器相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 概述

类加载器是用来加载字节码文件的，其将字节码文件加载到JVM中，将字节码中类静态存储结构转化为运行时数据结构存储在方法区中，并在堆中创建特殊对象Class对象来封装其在方法区的的数据，通过Class对象可以访问到方法区的数据，Class对象的类为java.lang.Class类。

### ClassLoader类

java.lang.ClassLoader是类加载器的类。

##### ClassLoader常用方法

|方法|说明|
|:----|:----|
|loadClass|加载名为name的字节码文件|
|findClass|查找名为name的字节码文件|
|findLoadedClass|查找名为name的已加载过的Class对象|
|defineClass|将参数中的数组转化为Class对象|
|resolveClass|链接指定的Class对象|
|getParent|获取当前ClassLoader的父ClassLoader|

loadClass是一个递归向上调用的过程，其先用findLoadedClass判断本加载器是否已加载过该类，如加载过，则根据传参判断是否需要调用resolveClass，然后返回对应Class对象，如无加载，且父类加载存在，则调用父类加载器的loadClass方法，如果父类加载器不存在，则调用findClass去加载类，如果成功，则根据传参判断是否需要调用resolveClass，然后返回对应Class对象，如果失败，则调用其子类加载器加载，如果最底层的类加载器仍未加载成功，则抛出ClassNotFoundException。

### 主要类加载器

* Bootstrap Class Loader

引导类加载器，其用来加载Java的核心库，例如JAVA_HOME下的lib类库，该加载器加载的目录可以由-Xbootclasspath来添加，其不继承java.lang.ClassLoader，其是用C++语言写的。

* Extension Class Loader

扩展类加载器，其用来加载Java的扩展库，例如JAVA_HOME下的lib/ext类库，该加载器加载的目录可以由-Djava.ext.dirs来添加，其继承了java.lang.ClassLoader类。

* Application Class Loader

应用类加载器，其用来加载应用程序的类路径CLASSPATH，该加载器加载的目录可以由-classpath，-cp来添加，是Java程序默认的类加载器，其继承了java.lang.ClassLoader类。

虚拟机启动时，会先初始化引导类加载器，然后在Launcher（JRE中用于启动main方法的类）中加载扩展类加载器和应用类加载器，并设置线程上下文类加载器。

### 主要类加载器层次关系

应用类加载器的父类加载器是扩展类加载器，扩展类加载器的父类加载器是引导类加载器。

自定义类加载器的父类加载器是应用类加载器。

### 类加载器隔离

每个类加载器都有自己的命名空间来存储已加载的类。

JVM判断两个类对象是否相等是判断类对象的全名称（包名+类名）和其类加载器是否相等。

全名称相同但其类加载器不同，则两个类对象不相等。

### 双亲委派模型

类加载器使用了双亲委派模型，其在加载类时，先会把该类传给其父类加载器去加载，如果其父类加载器还有父类加载器的话，继续往上传，当父加载器无法完成该类的加载，子加载器才执行，由上至下加载。

双亲委派模式保证了安全性（核心API不被覆盖），避免了重复加载类。

### 类加载器加载类

当类第一次使用（类的静态成员被程序引用）时，其会加载类，类的构造器也是类的静态方法。

##### 加载字节码流程

* 装载-Load

查找并加载类字节码文件

* 链接-Link

验证：验证加载的类字节码是否合法

准备：为类的静态变量分配内存，并初始化默认值

解析：将类中的符号引用装换为直接引用

* 初始化-Initialize

为类的静态变量初始化正确的值

##### 加载字节码方式

* 显式加载

通过代码显示的调用加载

* 隐式加载

通过虚拟机自动加载

##### 加载字节码途径

* 本地文件

* 网络

* zip，jar包

* 专有数据库

### 自定义类加载器

继承ClassLoader类。

扩展类加载器和应用类加载器是Launcher的静态内部类，是包访问路径权限的。

MyClassLoader

``` java
public class MyClassLoader extends ClassLoader {

    private String defaultClassLoaderPath;

    public MyClassLoader(String defaultClassLoaderPath) {
        this.defaultClassLoaderPath = defaultClassLoaderPath;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {

        try {
            InputStream fileInputStream = new FileInputStream(defaultClassLoaderPath + File.separator + name + ".class");
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];
            int length = 0;
            while ((length = fileInputStream.read(buffer)) != -1) {
                byteArrayOutputStream.write(buffer, 0, length);
            }
            return defineClass(name, byteArrayOutputStream.toByteArray(), 0, byteArrayOutputStream.size());
        } catch (Exception e) {
            e.printStackTrace();
            return defineClass(name, new byte[]{}, 0, 0);
        }
    }
}
```

MyClass

``` java
public class MyClass {

    public void say() {
        System.out.println("Hello World");
    }
}
```

将上述类通过javac命令编译成字节码文件

``` java
public static void main(String[] args) {

    try {
        MyClassLoader myClassLoader = new MyClassLoader("D:\\");
        Object myClass = myClassLoader.loadClass("MyClass").newInstance();
        myClass.getClass().getMethod("say").invoke(myClass);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

输出

``` text
Hello World
```

### 线程上下文类加载器

线程上下文类加载器（ContextClassLoader）是在JDK1.2时引入的，我们可以将类加载器存放到当前线程中，以方便我们使用对应的类加载器，但是这违背了双亲委派模型，如果没有显示设置线程的上下文类加载器，线程会继承其父线程的上下文类加载器。

##### SPI

服务提供接口（Service Provider Interface），Java中存在很多服务提供者接口（例如JDBC，JNDI等），这些接口允许第三方实现，但是这些接口属于Java核心库，一般存放于rt.jar中，是由引导类加载器加载的，第三方实现的代码实际使用中是作为Java程序所需的依赖存放在classpath路径下，classpath路径是由应用类加载器加载的，但是如果这样的话会导致无法使用，因为接口和实现类不是同一个加载器加载的，其不是同一个类型，所以上下文类加载器可以用来解决这个问题，将引导类加载器设置为线程上下文类加载器，然后用线程上下文类加载器加载。

### Class.forName

Class.forName是一个静态方法，其可以用来加载类。