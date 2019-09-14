### 数组

数组是由相同类型的基本类型或对象组成的序列。

### 定义数组

##### 数组标识符

``` java
int[] a;
int b[];
```

第一种格式表达上更加合理，int[]代表a的类型，第二种格式符合c和c++语言的习惯。

##### 数组初始化

``` java
int[] a = new int[5];
IntStream.rangeClosed(1, 5).forEach(i -> a[i - 1] = i);
int[] b = new int[]{1, 2, 3, 4, 5};
int[] c = {1, 2, 3, 4, 5};
```

### 特点

* 在java中，数组的存储和随机访问效率高。

* 数组的长度初始化时固定，在其整个生命周期中不可以改变。

* 数组中的元素在内存中是连续存放的，其标识符是一个引用，指向堆内的数组对象，数组对象保存指向其他对象的引用（如果是基本类型数组，其直接保存基本类型的值），length是数组中唯一一个可以被访问的，其表示数组的长度。

* 数组无法确定其有多少个有效数据，其在初始化时会对所有元素进行初始化赋值，基本类型中，数值型默认值为0，字符型默认值为空，布尔型默认值为false，对象类型数组，默认值为null。

* 数组有越界检查及类型检查，其数组中的数据类型需保持一致。

* 数组可以持有基本类型。

* 数组有最大长度限制。

### 多维数组

多维数组是数组里的元素同样也为数组，数组嵌套数组，从而形成了多个维度。

### 数组实用功能

##### 填充数组

Arrays.fill

``` java
int[] a = new int[5];
Arrays.fill(a, 1);
```

##### 复制数组

System.arraycopy

``` java
int[] a = new int[5];
Arrays.fill(a, 1);
int[] b = new int[5];
System.arraycopy(a, 0, b, 0, 5);
```

clone

``` java
int[] a = new int[5];
Arrays.fill(a, 1);
int[] b = new int[5];
b = a.clone();
```

Arrays.copyOf

``` java
int[] a = new int[5];
Arrays.fill(a, 1);
int[] b = new int[5];
b = Arrays.copyOf(a, 5);
```

效率比较

System.arraycopy > clone > Arrays.copyOf > for循环

System.arraycopy是native方法，Arrays.copyOf是对System.arraycopy的封装。

##### 数组比较

``` java
int[] a = new int[5];
Arrays.fill(a, 1);
int[] b = new int[5];
Arrays.fill(b, 1);

System.out.println(a.equals(b));
System.out.println(Arrays.equals(a, b));
```

输出

``` text
false
true
```

多维数组可以用Arrays.deepEquals。

##### 数组排序

``` java
int[] a = new int[]{3, 55, 24, 8, 10};
String[] b = new String[]{"hi", "hello", "HI", "HELLO", "Hello"};

Arrays.sort(a);
Arrays.stream(a).forEach(System.out::println);
Arrays.sort(b);
Arrays.stream(b).forEach(System.out::println);
Arrays.sort(b, Collections.reverseOrder());
Arrays.stream(b).forEach(System.out::println);
```

对象类型数组，其元素对应的类要实现Comparable接口或者使用自定义Comparator实现类。

##### 查找数组内容

``` java
int[] a = new int[]{3, 55, 24, 8, 10};

Arrays.sort(a);
int index = Arrays.binarySearch(a, 10);
System.out.println(a[index]);
```

输出

``` text
10
```

操作的数组先要排序，如果使用了Comparator进行排序，Arrays.binarySearch需将对应的Comparator传入。