### 概述

执行计划是用来查看DQL在DBMS中的执行情况。

### 语句

EXPLAIN + DQL

##### 字段说明

* id

表示执行顺序，id大的先执行，id一致则按照从上至下的顺序执行。

* select_type

|名称|说明|
|:----:|:----:|
|SIMPLE|简单查询，不使用UNION和子查询|
|PRIMARY|存在子查询时最外层的查询|
|UNION|UNION关键字后的SQL|
|SUBQUERY|非FROM关键字后的子查询|
|DERIVED|FROM关键字后的子查询|

* table

查询的表

* partition

查询的分区

* type

按效率从高到低排序

|名称|说明|
|:----:|:----:|
|system|表中只有一条数据（相当于系统表）|
|const|只匹配一条数据|
|eq_ref|唯一性索引，每个索引值只有一条数据匹配，常见于主键和唯一索引|
|ref|非唯一性索引|
|range|根据索引查找范围|
|index|只查找索引树|
|ALL|全表查找|

* possible_keys

可以查找到数据的索引

* key

实际使用的索引

* key_len

实际使用索引的长度

* ref

索引字段的关联字段

* rows

预计数据的行数

* filtered

匹配数据占查找数据的比例

* Extra

附加信息