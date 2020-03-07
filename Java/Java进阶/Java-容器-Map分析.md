### HashMap

HashMap使用了数组链表加红黑树，其用Node数组来存储链表（Node实现了Map.Entry）。

HashMap会根据key的hash来确定数组中的位置，如果数组该位置已有Node节点，则添加到对应位置的链表中，如果链表的长度大于8，会将链表转化为红黑树（JDK1.8新增的功能）。

如果链表数组的填装因子大于0.75(默认值)，则对数组进行扩容，数组初始化默认长度为16，扩容是开销较大的操作，扩容后的数组长度是原来的两倍。

HashMap允许key为null，key为null时其hash值按0处理，value也可以为null。

HashMap不是线程安全的。

### LinkedHashMap

LinkedHashMap继承了HashMap，其实现了有序的HashMap。

LinkedHashMap有序可以指的是插入顺序，也可以指的是是访问顺序，默认是插入顺序（accessOrder为false）。

LinkedHashMap定义了自己的Entry来存储数据，Entry继承了HashMap.Node，其在HashMap.Node的基础上额外存储了前一个Entry和后一个Entry，在HashMap的结构上添加了双向链表。

LinkedHashMap还存储了其双向链表的头和尾。

LinkedHashMap不是线程安全的。

### WeakHashMap

HashMap使用了数组链表，其用Entry数组来存储链表（Entry实现了Map.Entry）。

Hashtable会根据key的hash来确定数组中的位置，如果数组该位置已有Node节点，则添加到对应位置的链表中。

如果链表数组的填装因子大于等于0.75(默认值)，则对数组进行扩容，数组初始化默认长度为16，扩容是开销较大的操作，扩容后的数组长度是原来的两倍。

WeakHashMap中的key使用了弱引用，当key只有弱引用时，在下次gc时其会被回收，在操作一些方法时，其会通过expungeStaleEntries方法将被回收key对应的Entry从相应链表中移除，同时也会将该key对应的value置为null以帮助回收，size，get，put，remove方法会调用expungeStaleEntries方法（size方法在size为0时不会调用该方法）。

WeakHashMap允许key为null，key为null时其hash值按WeakHashMap中的NULL_KEY静态成员（new Object()）的hash值处理，value也可以为null。

WeakHashMap不是线程安全的。

### Hashtable

Hashtable使用了数组链表，其用Entry数组来存储链表（Entry实现了Map.Entry）。

Hashtable会根据key的hash来确定数组中的位置，如果数组该位置已有Node节点，则添加到对应位置的链表中。

如果链表数组的填装因子大于等于0.75(默认值)，则对数组进行扩容，数组初始化默认长度为11，扩容是开销较大的操作，扩容后的数组长度是原来的两倍加一。

Hashtable不允许key和value为null。

Hashtable是线程安全的，其方法使用了synchronized修饰，但其性能不如ConcurrentHashMap，不推荐使用。

### ConcurrentHashMap

ConcurrentHashMap使用了数组链表加红黑树，其用Node数组来存储链表（Node实现了Map.Entry）。

ConcurrentHashMap会根据key的hash来确定数组中的位置，如果数组该位置已有Node节点，则添加到对应位置的链表中，如果链表的长度大于8，会将链表转化为红黑树（JDK1.8新增的功能）。

如果链表数组的填装因子大于等于0.75(默认值)，则对数组进行扩容，数组初始化默认长度为16（添加第一个元素时才为16，初始化时，数组为空），扩容是开销较大的操作，扩容后的数组长度是原来的两倍。

ConcurrentHashMap不允许key和value为null。

ConcurrentHashMap是线程安全的，且性能优于Hashtable，JDK1.8采用了CAS+synchronized来保证并发更新。

### TreeMap

TreeMap使用了红黑树，红黑树的节点为Entry（Entry实现了Map.Entry）。

默认情况下会按key的自然顺序升序排序，也可以指定排序器。

TreeMap不允许key和value为null。

### 补充说明

Hash方式的Map使用key的Hash值作为存储位置的判断依据，所以key值是有要求的，作为key值的对象需要保证equals方法相等时，其hash值也一样，一般情况下使用String来作为key。