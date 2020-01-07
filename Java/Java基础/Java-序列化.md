### 概述

序列化是将对象转化为字节序列的过程，反序列化是将字节序列转化为对象的过程。

序列化和反序列化在对象存储和传输的时候使用。

### Java序列化

示例

``` java
@Data
public class Person implements Serializable {

    private static final long serialVersionUID = -8309107389571419210L;
    private String name;
    private int age;
}
```

``` java
Person personOne = new Person();
personOne.setName("小白");
personOne.setAge(20);

FileOutputStream fileOutputStream = new FileOutputStream("./record.txt");
ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);
objectOutputStream.writeObject(personOne);
objectOutputStream.close();
fileOutputStream.close();

FileInputStream fileInputStream = new FileInputStream("./record.txt");
ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream);
Person personTwo = (Person) objectInputStream.readObject();
objectInputStream.close();
fileInputStream.close();

System.out.println(personTwo.getName() + "-" + personTwo.getAge());
```

``` text
小白-20
```

* serialVersionUID可以自动生成，也可以手动设置，反序列化对象类的serialVersionUID和序列化对象类的serialVersionUID一致才可以反序列化成功，自动生成的serialVersionUID受类的属性影响，当类的属性改变时，其再生成的serialVersionUID也会变化。

* Serializable不支持static修饰的属性。

* transient修饰的属性序列化会忽略。

### 自定义序列化

自定义序列化需要实现Externalizable接口。

writeExternal方法需要将对象中的数据序列化后写入ObjectOutput。

readExternal方法需要从ObjectInput读取序列化后的数据并还原对象。

### 其他常见序列化方式

Json，Xml。