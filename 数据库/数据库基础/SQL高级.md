### 视图

视图是虚拟的表，其是对SELECT语句结果的封装，从而简化复杂的SQL。

##### 视图操作

``` sql
CREATE OR REPLACE VIEW student_score AS
SELECT student.student_name,
(SELECT AVG(score.score) FROM score WHERE score.student_no = student.student_no) AS avg_score
FROM student;

CREATE VIEW student_score AS
SELECT student.student_name,
(SELECT AVG(score.score) FROM score WHERE score.student_no = student.student_no) AS avg_score
FROM student;
```

视图名不能重复，也不能和表名一样。

一些DBMS禁止视图中使用GROUP BY子句。

视图不能索引。

``` sql
SELECT * FROM student_score;
```

``` sql
CREATE OR REPLACE VIEW student_score AS
SELECT student.student_no, student.student_name,
(SELECT AVG(score.score) FROM score WHERE score.student_no = student.student_no) AS avg_score
FROM student;

ALTER VIEW student_score AS
SELECT student.student_no, student.student_name,
(SELECT AVG(score.score) FROM score WHERE score.student_no = student.student_no) AS avg_score
FROM student;
```

``` sql
DROP VIEW student_score;
```

### 储存过程

存储过程是封装在数据库中为了完成特定功能的SQL语句集。

##### 存储过程优缺点

存储过程实现了功能的复用，提高了安全性，减少了网络带宽（调用次数少，传输信息少），提高了执行效率（存储过程在创建的时候就已编译了，其在使用过程中不需要再编译）。

存储过程编写较复杂，不同DBMS的语法不一致，移植性不好。

### 事务

事务是数据库数据操作的一个基本单位，其包含一条或多条数据库数据操作。

不同的DBMS实现的语法有所不同。

SET TRANSACTION可以设置事务的隔离级别。

* COMMIT-提交

``` sql
BEGIN;
INSERT INTO student (student_no, student_name, student_age, student_sex) VALUES ('2019001001', '小黑', 18, '00');
INSERT INTO student (student_no, student_name, student_age, student_sex) VALUES ('2019001002', '小白', 20, '01');
COMMIT;
```

BEGIN和START TRANSACTION等价。

COMMIT和COMMIT WORK等价。

* ROLLBACK-回退

``` sql
BEGIN;
INSERT INTO student (student_no, student_name, student_age, student_sex) VALUES ('2019001001', '小黑', 18, '00');
INSERT INTO student (student_no, student_name, student_age, student_sex) VALUES ('2019001002', '小白', 20, '01');
ROLLBACK;
```

ROLLBACK和ROLLBACK WORK等价。

* SAVEPOINT-保存点

``` sql
BEGIN;
INSERT INTO student (student_no, student_name, student_age, student_sex) VALUES ('2019001001', '小黑', 18, '00');
SAVEPOINT a;
INSERT INTO student (student_no, student_name, student_age, student_sex) VALUES ('2019001002', '小白', 20, '01');
ROLLBACK TO a;
COMMIT;
```

``` sql
--删除保存点
RELEASE SAVEPOINT a;
```

### 游标

游标是用来单条操作SELECT结果集中数据的。

不同的DBMS实现的语法有所不同。

游标是应用在存储过程中的。

### 约束

约束是指用来规范数据库中数据的规则。

##### 常见约束

* NOT NULL

不能为空。

* UNIQUE

唯一限制。

* PRIMARY KEY

主键限制。

* FOREIGN KEY

外键限制，外键是指其必须为另一个表的主键。

* CHECK

检查数据是否满足规则。

* DEFAULT

默认值。

### 索引

索引类似书的目录，其可以加快数据库查询数据的效率，但是同时也降低了数据插入，修改和删除的效率。

索引可能占用大量的空间。

索引名要唯一，索引也可以作用于多个列。

### 触发器

触发器是特殊的存储过程，其在数据库发生相应操作时自动执行。