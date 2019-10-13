### MySQL概述

MySQL是一种流行的关系型DBMS，其是服务端-客户端的模式。

MySQL是由瑞典MySQL AB公司开发，现属于Oracle。

Mysql体积小，开源免费，速度快。

### Mysql存储引擎

Mysql可以自由的选择存储引擎，存储引擎是数据库底层组件，数据库管理系统使用数据引擎来管理操作数据，常见的储存引擎有InnoDB，MyISAM，MEMORY，Archive等。

``` sql
-- 可以查看数据库支持的存储引擎
SHOW ENGINES;
```

##### InnoDB

* MySQL5.5版本后的默认存储引擎。

* 支持ACID事务，支持外键，支持行级锁。

* 存储限制为64TB

* 不支持全文索引，支持树索引

##### MyISAM

* MySQL官方存储引擎，5.5版本之前的默认存储引擎。

* 不支持事务，表级锁，查询效率高。

* 存储限制为256TB

* 支持全文索引，支持树索引

##### MEMORY

* 临时存放数据，且数据量不大，对数据的安全性要求也不高

* 不支持事务

* 存储在内存中，存储受内存限制

* 不支持全文索引，支持树，哈希索引。

##### Archive

* 只支持插入和查询操作，支持高并发的插入

* 不支持事务

* 存储无限制

* 不支持索引

### MySQL锁

* 表级锁

开销小，加锁快，可以避免死锁，但是锁冲突概率大，并发效率低。

* 行级锁

开销大，加锁慢，会发生死锁，但是锁冲突概率小，并发效率高。

* 页面锁

开销介于表级锁和行级锁之间，加锁介于表级锁和行级锁之间，会发生死锁，锁冲突概率介于表级锁和行级锁之间，并发效率介于表级锁和行级锁之间。

### MySQL基本SQL

* 查看服务器状态信息

``` sql
SHOW STATUS;
```

* 查看用户权限信息

``` sql
SHOW GRANTS;
```

* 显示数据库

``` sql
SHOW DATABASES;
```

* 显示创建数据库语句

``` sql
SHOW CREATE DATABASE study;
```

* 选择数据库

``` sql
USE study;
```

* 显示当前库的表

``` sql
SHOW TABLES;
```

* 显示创建表语句

``` sql
SHOW CREATE TABLE student;
```

* 显示表的列

``` sql
DESCRIBE student;
SHOW COLUMNS FROM student;
```

* 查询指定行数数据

``` sql
-- 行数标识从0开始
-- 前两行
SELECT * FROM score ORDER BY student_no LIMIT 2;
-- 从第三行开始（包含第三行）的两行数据
SELECT * FROM score ORDER BY student_no LIMIT 2,2;
SELECT * FROM score ORDER BY student_no LIMIT 2 OFFSET 2;
```

* 完全限定

``` sql
SELECT student.student_no, student.student_name FROM study.student;
```

* 时间处理

``` sql
-- 23:46:20
SELECT CURTIME();
SELECT CURRENT_TIME;
SELECT CURRENT_TIME();
-- 2019-10-12
SELECT CURDATE();
SELECT CURRENT_DATE;
SELECT CURRENT_DATE();
-- 2019-10-12 23:47:27
SELECT NOW();
SELECT SYSDATE();
```

* 自增

AUTO_INCREMENT可以设置一个列为自增列，每次插入新数据时，该列的值会自己增长，每个表只能有一个自增列，且该列必须有索引。