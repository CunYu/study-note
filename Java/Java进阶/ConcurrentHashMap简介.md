### 概述

ConcurrentHashMap是Java中线程安全的HashMap，因为Hashtable通过使用synchronized修饰方法的方式来保证线程安全，所以其性能并不高，ConcurrentHashMap相对比Hashtable，其性能更高。

ConcurrentHashMap在Java 1.7和1.8版本中的实现方式有所不同。

### Java 1.7

ConcurrentHashMap采用了分段锁的方式，其将HashMap分为多个Segment，每个Segment的结构和HashMap一致，Segment也实现了ReentrantLock，所以Segment是一个锁，其使得一个Segment同时只能被一个线程写，读是不加锁的。

ConcurrentHashMap实际上是采用了拆分HashMap的方式，其将HashMap拆分成了一个个的小HashMap，每个小HashMap都会有自己的锁。不同的小HashMap是可以同时写处理的。

ConcurrentHashMap由于使用Segment，所以其效率没有HashMap高，因为其要进行根据Hash值进行两次确定位置（在那个Segment，在Segment中的那个位置）操作。

ConcurrentHashMap获取长度时，先不对Segment加锁，其会统计两次，如两次统计结果不一致，其会再进行统计，如果超过了阀值其会对Segment加锁，然后再重新进行统计。

### Java 1.8

ConcurrentHashMap采用自旋加CAS加synchronized的方式来保证线程安全。

ConcurrentHashMap部分源码

``` java
/**
 * Maps the specified key to the specified value in this table.
 * Neither the key nor the value can be null.
 *
 * <p>The value can be retrieved by calling the {@code get} method
 * with a key that is equal to the original key.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with {@code key}, or
 *         {@code null} if there was no mapping for {@code key}
 * @throws NullPointerException if the specified key or value is null
 */
public V put(K key, V value) {
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                            new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                    (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                            value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                        value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```