本文简单介绍了redis相关知识及java连接redis的demo。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### Redis 概述

Redis是一个将数据存储在内存中的key-value存储系统。可以用作数据库，缓存，消息中间件。

### Redis 特性

Redis读写速度十分快。

Redis支持持久化，可以将数据存储到硬盘中。

Redis支持多种数据结构存储。

Redis支持主从同步。

Redis是线程安全的，所有的操作都是原子性的。其主要功能都是基于单线程模型实现的，使用一个线程处理所有客户端请求。

Redis支持事务。

### Redis Key-Value

Redis的基础数据类型为string，所有的基本类型在redis中，都以string类型存储和操作。

Redis中key和value的支持长度最大为512MB。

Key支持的字符数据类型。

Value支持字符串（strings），哈希（hashes），列表（lists），集合（sets），有序集合（sorted sets）等。

### Redis 持久化

##### RDB

RDB持久化会定期将数据持久化到硬盘。

##### AOF

AOF会将redis的所有写命令记录在硬盘中。

### Redis 事务

Redis事务保证了事务内的所有命令都是绝对连续执行的，中间不会有其他的命令插入，redis不支持事务回滚。

### Redis 常见命令

##### 字符串（strings）

```
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

```
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

```
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

```
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

```
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

```
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

```
Jedis jedis = new Jedis("192.168.99.100");
jedis.set("demo", "2");
System.out.println(jedis.get("demo"));
jedis.close();
```

```
2
```

连接池连接

```
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

```
2
```