### 概述

TopK算法是找出计算数据中最大或最小的k个数。

### 时间复杂度

TopK算法的时间复杂度为O(nlog<sub>2</sub>k)。

### 说明

* 最大的k个数

将数据的前k个数建立一个最小堆，之后的每个数都与最小堆的根结点比较，若大于，则替换最小堆的根节点，然后调整堆，最小堆中的k个数极为数据中最大的k个数。

* 最小的k个数

将数据的前k个数建立一个最大堆，之后的每个数都与最大堆的根结点比较，若小于，则替换最小堆的根节点，然后调整堆，最大堆中的k个数极为数据中最小的k个数。

* 大数据处理

如果数据量很大时，可以将数据分为多份，每份都找出TopK值对应的堆，然后再对这些堆中的数据合集进行TopK算法。

### 示例

``` java
public static void main(String[] args) {

    int capacity = 100000;
    List<Integer> data = new ArrayList<>(capacity);
    IntStream.range(0, capacity).forEach(i -> {
        data.add((int) (Math.random() * capacity));
    });
    topTen(data).stream().forEach(System.out::println);
}

public static List<Integer> topTen(List<Integer> data) {

    PriorityQueue<Integer> priorityQueue = new PriorityQueue<>();
    data.stream().forEach(i -> {
        int length = priorityQueue.size();
        if (10 > length) {
            priorityQueue.offer(i);
        } else {
            Integer min = priorityQueue.poll();
            if (i > min) {
                priorityQueue.offer(i);
            } else {
                priorityQueue.offer(min);
            }
        }
    });

    Integer integer;
    List<Integer> topTenList = new ArrayList<>(10);
    while ((integer = priorityQueue.poll()) != null) {
        topTenList.add(integer);
    }
    return topTenList;
}
```

``` text
99991
99991
99992
99993
99996
99996
99996
99997
99998
99999
```