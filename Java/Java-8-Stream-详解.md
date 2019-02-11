本文介绍了Java 8 Stream流。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### Stream 说明

Stream是Java 8 新引入的概念，其提供了一种新的操作数据的方式，使操作数据更加的简单且高效。

Stream是一个支持顺序和并行聚合操作的元素序列。

### Stream Pipeline 说明

Stream在计算过程中，stream operation需要用到stream pipeline，stream pipeline会将stream operation按顺序记录下来。

### Stream Pipeline 组成

##### Source

一个stream pipeline包含一个source。

Source为流的来源。

##### Intermediate Operation

一个stream pipeline包含零个或多个intermediate operation。

Intermediate operation会返回一个新的流。

Intermediate operation是惰性化的，不会立即执行。

Intermediate opertaionf分为stateless operation和stateful operation。

Staleless operation中的每个元素都是可以独立操作的，不需要依赖其他元素。例如：filter()，map()。

Staleful operation中元素的处理依赖已处理过的元素。例如：distinct()，sorted()。

##### Terminal Operation

一个stream pipeline包含一个terminal operation。

terminal operation会产生一个result或者side-effect。

Terminal operation大多数是实时的,是立即执行的，当terminal operation开始执行时，interminal operation才开始执行，在返会result或者side-effect前会处理完所有的interminal operaion。只有iterator()和spliterator()是例外，这俩个操作是在当前现有功能不能满足预期的效果时，允许自定义操作流。

### Short-Circuiting

Short-Circuiting是在另一个维度描述operation，如果需要在有限的时间里处理无限元素的数据源时，short-circuiting operation是必要的。

* 在intermediate operation中，short-circuiting operation接收一个无限元素的流，返回一个有限元素的新流。

* 在terminal operation中，short-circuiting operation接收一个无限元素的流，会在有限的时间内产生一个result或者side-effect。

### Stream 注意事项

* Stream是不需要我们在使用后关闭stream的，除非source为IO channel的stream。

* Stream是可消耗的，在stream生命周期中，stream中的元素只能被访问一次。

* Stream只能进行一次operation，如果进行多次operation，有可能会抛出IllegalStateException异常。

### Stream Operation 理解说明

* Stream operation可以理解对source的“查询”，不会去修改source。除非专门为并发修改设计的数据源（例如ConcurrentHashMap）,否则在operation中修改source会出现错误或者不可预知的结果。

* Stream operation大多数的参数是描述用户指定行为的，大多数的参数为lamada表达式

* Stream operaion的结果不会受到任何在执行中可能改变的状态的影响。

### Stream 串行有序，并行模式

Stream支持串行有序，并行模式，这是Stream的属性之一。

流可以在创建的时候选择串行有序或者并行模式。例如stream()创建串行有序流，parallelStream()创建的是并行流。

流也可以通过sequential()，parallelStream()方法去改变流的模式。

流的模式可以通过sequential()方法判断。

流的串行有序，并行模式不会改变流的结果，除一些不确定结果的operation，例如findAny()。

Stream的并行用到了Fork/Join 框架。

Stream并行线程数默认为当前机器的处理核心数减一。例如，四核八线程机器，stream并行默认7个线程。Stream并行线程数可以通过System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "3")去设置。

并行模式不一定比串行有序模式快，要根据实际使用情况去选择串行有序模式，还是并行模式。

### Stream 高效性

Stream 的惰性化大大提升了Stream的效率，stream pipeline中，流的多次intermediate operation实际上最后会新形成一个最小的中间状态去操作一次source。

流的惰性操作还允许在不必要的情况下，就不需要遍历所有数据。

### Stream Collection 区别

* Stream没有存储数据的功能，Collection有存储数据的功能。

* Stream的侧重点在如何声明source以及相应的operation。

* Collection的侧重点在如何高效的管理，访问元素。

* Stream中元素可以是无限的，Collect中元素是有限的。

### Stream 常用构建
  
包装类型（减少装箱，拆箱消耗）
DoubleStream, IntStream, LongStream

非包装类型
Stream<T>

* 数组

``` java
String[] letters = {"a", "b", "c"}; 

Stream streamOne = Arrays.stream(letters);
Stream streamTwo = Stream.of(letters);
```

* List

``` java
List<String> animals = new ArrayList<>();
animals.add("dog");
animals.add("cat");
animals.add("chicken");

Stream stream = animals.stream();
```

### Stream 常用Operation

* forEach

``` java
List<String> animals = new ArrayList<>();
animals.add("dog");
animals.add("cat");
animals.add("chicken");

animals.stream().forEach(System.out::println);

IntStream.range(1, 2).forEach(System.out::println);
IntStream.rangeClosed(1, 2).forEach(System.out::println);
```

```
dog
cat
chicken
1
1
2
```

* filter

``` java
List<String> animals = new ArrayList<>();
animals.add("dog");
animals.add("cat");
animals.add("chicken");

animals.stream().filter(animal -> !"chicken".equals(animal)).forEach(System.out::println);
```

```
dog
cat
```

* map

``` java
List<String> animals = new ArrayList<>();
animals.add("dog");
animals.add("cat");
animals.add("chicken");

animals.stream().map(animal -> "hello " + animal).forEach(System.out::println);
```

```
hello dog
hello cat
hello chicken
```

* distinct

``` java
List<String> animals = new ArrayList<>();
animals.add("dog");
animals.add("cat");
animals.add("cat");
animals.add("chicken");

animals.stream().distinct().forEach(System.out::println);
```

```
dog
cat
chicken
```

* sorted

``` java
List<String> animals = new ArrayList<>();
animals.add("dog");
animals.add("cat");
animals.add("chicken");

animals.stream().sorted((a, b) -> a.compareTo(b)).forEach(System.out::println);
```

```
cat
chicken
dog
```

* sum

``` java
int[] numbers = {1, 2, 3};

int result = Arrays.stream(numbers).sum();
System.out.println(result);
```

```
6
```

* reduce

``` java
int[] numbers = {1, 2, 3};
String[] strArr = {"1", "2", "3"};

int resultOne = Arrays.stream(numbers).reduce(0, (a, b) -> a + b);
int resultTwo = Arrays.stream(numbers).reduce(0, Integer::sum);
BigDecimal resultThree = Arrays.stream(strArr).map(str -> new BigDecimal(str)).reduce(BigDecimal.ZERO, BigDecimal::add);

System.out.println(resultOne);
System.out.println(resultTwo);
System.out.println(resultThree);
```

```
6
6
6
```

* toArray

``` java
List<String> animals = new ArrayList<>();
animals.add("dog");
animals.add("cat");
animals.add("chicken");

String[] animalsArray = animals.stream().toArray(String[]::new);

Arrays.stream(animalsArray).forEach(System.out::println);
```

```
dog
cat
chicken
```

* collect

``` java
List<String> animals = new ArrayList<>();
animals.add("dog");
animals.add("cat");
animals.add("chicken");

List<String> animalsNewOne = animals.stream().collect(Collectors.toCollection(ArrayList::new));
List<String> animalsNewTwo = animals.stream().collect(Collectors.toList());
String animalsNewThree = animals.stream().collect(Collectors.joining(" "));

animalsNewOne.stream().forEach(System.out::println);
System.out.println("---");
animalsNewTwo.stream().forEach(System.out::println);
System.out.println("---");
System.out.println(animalsNewThree);
```

```
dog
cat
chicken
---
dog
cat
chicken
---
dog cat chicken
```

* Match

``` java
List<String> animals = new ArrayList<>();
animals.add("dog");
animals.add("cat");
animals.add("chicken");

System.out.println(animals.stream().allMatch(str -> str.startsWith("d")));
System.out.println(animals.stream().anyMatch(str -> str.startsWith("d")));
System.out.println(animals.stream().noneMatch(str -> str.startsWith("d")));
```

```
false
true
false
```
