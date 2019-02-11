本文介绍了java注解相关知识并附有相关的demo。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### Java 注解概述

Java注解即为java语言中的注解，注解又称为元数据，其可以将信息附加在代码中，相当于给代码打上标识。其在Java 1.5版本中引入。

### Java 常见标准注解

|注解|作用|
|:----|:----|
|@Override|该注解表明当前方法重写或实现了其超类中的方法。|
|@Deprecated|该注解表明当前方法不推荐使用，将来有可能被替换或遗弃。|
|@SuppressWarnings|该注解表明当前方法会忽略掉相应参数对应的编译器警告。|

### 元注解

元注解是用来修饰注解的注解。

|注解|作用|参数|
|:----|:----|:----|
|@Target|该注解声明注解位置。|详见ElementType枚举。|
|@Retention|该注解声明注解的生命周期。|详见RetentionPolicy枚举。|
|@Documented|该注解声明的注解显示在javadoc中。|无|
|@Inherited|该注解声明的注解可以被子类继承。|无|

|枚举|作用|
|:----|:----|
|ElementType.TYPE|类，接口，注解，枚举。|
|ElementType.FIELD|域（成员变量）声明，包含枚举实例。|
|ElementType.METHOD|方法|
|ElementType.PARAMETER|参数|
|ElementType.CONSTRUCTOR|构造器|
|ElementType.LOCAL_VARIABLE|局部变量|
|ElementType.ANNOTATION_TYPE|注解|
|ElementType.PACKAGE|包|
|ElementType.TYPE_PARAMETER|参数类型|
|ElementType.TYPE_USE|类型使用|
|RetentionPolicy.SOURCE|源代码|
|RetentionPolicy.CLASS|Class文件|
|RetentionPolicy.RUNTIME|内存中的字节码|

### 注解定义

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface MessageRecord {

    String code();
    String message() default "";
}
```

### 注解元素

注解定义中code和message为注解的元素。

注解元素只能为如下类型

* 所有的基本类型

* String

* Class

* enum

* Annotation

* 以上类型的数组

### 注解元素默认值

注解元素不允许为null，通常用负数或者空字符串表示空。

### 标记注解

不包含元素的注解称为标记注解。 

### 注解不支持继承

注解定义时不能使用extends，因为其本质上继承了Annotation接口。

### 自定义注解反射实现（运行时）

自定义注解

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface MessageRecord {

    String code();
    String message() default "";
}
```

Person类

```
public class Person {

    // 注解参数格式为key = value，如果该注解只有一个元素，则可以忽略key，只写value即可。
    @MessageRecord(code = "name")
    private String name;
    @MessageRecord(code = "sex", message = "性别")
    private String sex;
    @MessageRecord(code = "age", message = "年龄")
    private String age;
}
```

注解处理

```
Field[] fields = Person.class.getDeclaredFields();

for (Field field : fields) {

    if (field.isAnnotationPresent(MessageRecord.class)) {

        MessageRecord messageRecord = field.getAnnotation(MessageRecord.class);
        System.out.println(messageRecord.code() + ":" + messageRecord.message());
    }
}
```

输出

```
name:
sex:性别
age:年龄
```

### 自定义注解处理器实现（编译时）

自定义注解处理器运用了APT（Annotation Process Tool）。

注解处理器项目结构

```
src
-- main
---- src
------ java
-------- MessageRecord.java
-------- MessageRecordProcessor.java
------ resources
-------- META-INF
---------- services
------------ javax.annotation.processing.Processor
build.gradle
```

MessageRecord.java

```
import java.lang.annotation.*;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.CLASS)
@Documented
@Inherited
public @interface MessageRecord {

    String code();
    String message() default "";
}
```

MessageRecordProcessor.java

```
import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.Messager;
import javax.annotation.processing.ProcessingEnvironment;
import javax.annotation.processing.RoundEnvironment;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.TypeElement;
import javax.tools.Diagnostic;
import java.util.HashSet;
import java.util.Set;

public class MessageRecordProcessor extends AbstractProcessor {

    private Messager messager;

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        this.messager = processingEnv.getMessager();
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(MessageRecord.class);
        for (Element element : elements) {
            MessageRecord messageRecord = element.getAnnotation(MessageRecord.class);
            messager.printMessage(Diagnostic.Kind.NOTE, messageRecord.code() + ":" + messageRecord.message());
        }
        return true;
    }

    // 可用@SupportedAnnotationTypes({"MessageRecord"})替代
    @Override
    public Set<String> getSupportedAnnotationTypes() {
        Set<String> annotation = new HashSet<>();
        annotation.add(MessageRecord.class.getCanonicalName());
        return annotation;
    }

    // 可用@SupportedSourceVersion(SourceVersion.RELEASE_8)替代
    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }
}
```

javax.annotation.processing.Processor

存放注解处理器地址（包路径+类名），注册注解处理器。多个注解处理器时，每个注解处理器地址写完后需换行。

```
MessageRecordProcessor
```

build.gradle

```
apply plugin: 'java'
```

在项目下根目录下执行gradle build命令，在生成的文件中可以找到一个jar包，该jar包即为自定义注解处理器。

验证自定义注解处理器项目结构

```
src
-- main
---- src
------ java
-------- Person.java
libs
-- xxx.jar
build.gradle
```

Person.java

```
public class Person {

    @MessageRecord(code = "name")
    private String name;
    @MessageRecord(code = "sex", message = "性别")
    private String sex;
    @MessageRecord(code = "age", message = "年龄")
    private String age;
}
```

xxx.jar即为上文中自定义注解处理器jar包。

build.gradle

```
apply plugin: 'java'

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}

tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
}
```

在项目根目录下执行gradle build命令，在输出中即可看到相应的结果。

部分输出内容

```
注: name:
注: sex:性别                
注: age:年龄
```
