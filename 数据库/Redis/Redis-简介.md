### Redis 概述

Redis是一个将数据存储在内存中的key-value存储系统。可以用作数据库，缓存，消息中间件。

### Redis 特性

Redis性能十分出色，读写速度很快。

Redis支持多种数据结构存储。

Redis支持持久化，可以将数据存储到硬盘中。

Redis支持事务。

Redis支持集群搭建，可以主从同步。

Redis是线程安全的，所有的操作都是原子性的。其主要功能都是基于单线程模型实现的，使用一个线程处理所有客户端请求。

### Redis Key-Value

Redis的基础数据类型为string，所有的基本类型在redis中，都以string类型存储和操作。

Redis中key和value的支持长度最大为512MB。

Key支持字符串（string）类型。

Value支持字符串（string），哈希（hash），列表（list），集合（set），有序集合（sorted set）类型。

### Redis 持久化

##### RDB

RDB持久化会定期将数据持久化到硬盘，RDB也称为快照。

RDB在将数据持久化到硬盘时，如果数据较多时，性能开销很大，会间断性暂停服务。

##### AOF

AOF会将redis的所有写命令记录在硬盘中。

如果AOF文件较大时，在重启或根据AOF文件恢复数据数据时性能开销很大，会间断性暂停服务。

##### 说明

建议不要在主节点进行持久化，持久化可以放到从节点进行。

### Redis 事务

Redis事务保证了事务内的所有命令都是绝对连续执行的，中间不会有其他的命令插入，redis不支持事务回滚。

### Redis 回收策略

当Redis中的数据达到一定大小的时候，就会根据回收策略回收数据。

* volatile-lru 

从设置过期时间的数据集中选择最近最少使用的数据回收

* volatile-ttl

从设置过期时间的数据集中选择将要过期的数据回收

* volatile-random

从设置过期时间的数据集中随机选择数据回收

* allkeys-lru

从数据集中选择最近最少使用的数据回收

* allkeys-random

从数据集中随机选择数据回收

* no-enviction

禁止回收数据

### Redis 常见命令

##### 字符串（strings）

``` shell
127.0.0.1:6379> set demo "2"
OK
127.0.0.1:6379> get demo
"2"
127.0.0.1:6379> incr demo
(integer) 3
127.0.0.1:6379> get demo
"3"
127.0.0.1:6379> incrby demo 2
(integer) 5
127.0.0.1:6379> get demo
"5"
127.0.0.1:6379> append demo "0"
(integer) 2
127.0.0.1:6379> get demo
"50"
127.0.0.1:6379> del demo
(integer) 1
```

##### 哈希（hashes）

``` shell
127.0.0.1:6379> hmset demo key1 "value1" key2 "value2" key3 "value3"
OK
127.0.0.1:6379> hgetall demo
1) "key1"
2) "value1"
3) "key2"
4) "value2"
5) "key3"
6) "value3"
127.0.0.1:6379> hmget demo key1
1) "value1"
127.0.0.1:6379> hmget demo key1 key2
1) "value1"
2) "value2"
127.0.0.1:6379> del demo
(integer) 1
```

##### 列表（lists）

有序

``` shell
127.0.0.1:6379> lpush demo "1"
(integer) 1
127.0.0.1:6379> lpush demo "2" "3" "3"
(integer) 4
127.0.0.1:6379> llen demo
(integer) 4
127.0.0.1:6379> lindex demo 2
"2"
127.0.0.1:6379> lrange demo 0 3
1) "3"
2) "3"
3) "2"
4) "1"
127.0.0.1:6379> lrange demo 0 -1
1) "3"
2) "3"
3) "2"
4) "1"
127.0.0.1:6379> rpush demo "4" "5"
(integer) 6
127.0.0.1:6379> lrange demo 0 5
1) "3"
2) "3"
3) "2"
4) "1"
5) "4"
6) "5"
127.0.0.1:6379> lindex demo 0
"3"
127.0.0.1:6379> lindex demo 3
"1"
127.0.0.1:6379> del demo
(integer) 1
```

##### 集合（sets）

无序

``` shell
127.0.0.1:6379> sadd demo "1"
(integer) 1
127.0.0.1:6379> sadd demo "2"
(integer) 1
127.0.0.1:6379> sadd demo "3"
(integer) 1
127.0.0.1:6379> scard demo
(integer) 3
127.0.0.1:6379> sismember demo "2"
(integer) 1
127.0.0.1:6379> sismember demo "4"
(integer) 0
127.0.0.1:6379> smembers demo
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> del demo
(integer) 1
```

##### 有序集合（sorted sets）

有序

以score进行排序

``` shell
127.0.0.1:6379> zadd demo 1 "one"
(integer) 1
127.0.0.1:6379> zadd demo 3 "three"
(integer) 1
127.0.0.1:6379> zadd demo 2 "two"
(integer) 1
127.0.0.1:6379> zcard demo
(integer) 3
127.0.0.1:6379> zrange demo 0 -1 withscores
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"
127.0.0.1:6379> zrange demo 0 -1
1) "one"
2) "two"
3) "three"
```

### Jedis

Jedis是redis的java连接工具之一。

build.gradle

``` groovy
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile('redis.clients:jedis:2.9.0')
}
```

jedis demo

普通连接

``` java
Jedis jedis = new Jedis("192.168.99.100");
jedis.set("demo", "2");
System.out.println(jedis.get("demo"));
jedis.close();
```

``` text
2
```

连接池连接

``` java
JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
jedisPoolConfig.setMaxTotal(50);
jedisPoolConfig.setMinIdle(10);
JedisPool jedisPool = new JedisPool(jedisPoolConfig, "192.168.99.100");

Jedis jedis = jedisPool.getResource();
// 如果存放对象，需将对象序列化放入，取出时再反序列化还原对象
jedis.set("demo", "2");
System.out.println(jedis.get("demo"));
jedis.close();
jedisPool.close();
```

``` text
2
```