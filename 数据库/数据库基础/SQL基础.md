### 表的管理

##### 创建表

``` sql
CREATE TABLE student (
    student_no      CHAR(10)    NOT NULL,
    student_name    CHAR(64)    NOT NULL,
    student_age     INT         NULL,
    student_sex     CHAR(2)     NOT NULL,
    student_grade   INT         NOT NULL    DEFAULT 1
);

CREATE TABLE score (
    student_no      CHAR(10)    NOT NULL,
    subject         CHAR(50)    NOT NULL,
    score           INT         NOT NULL
);
```

##### 修改表

``` sql
ALTER TABLE student ADD student_remark CHAR(64);
ALTER TABLE student DROP COLUMN student_remark;
```

##### 删除表

``` sql
DROP TABLE student;
```

### 数据操作

##### 插入

* 直接插入

``` sql
INSERT INTO student VALUES ('2019001001', '小黑', 18, '00', 1);
INSERT INTO student (student_no, student_name, student_age, student_sex) VALUES ('2019001002', '小白', 20, '01');
```

插入时建议使用给出列名的方法。

* 插入查询到的数据

``` sql
INSERT INTO student_new (student_no, student_name, student_age, student_sex)
    SELECT student_no, student_name, student_age, student_sex FROM student;
```

将查询到的数据插入student表中，列名称不一样也可以，数据匹配即可。

INSERT语句一般是一条语句插入一行，该情况是个例外，一条语句插入多行。

##### 查询

* 基本查询

``` sql
SELECT * FROM student;
SELECT student_no, student_name FROM student;
```

一般情况下，不建议使用通配符，查询不需要的列会降低性能。

* 检索不重复列

``` sql
SELECT DISTINCT student_age FROM student;
```

DISTINCT是作用于所有列的，如果有2个列，则去重是去掉两个列数据都相同的。

* 排序

``` sql
SELECT student_no, student_name FROM student ORDER BY student_no;
SELECT student_no, student_name FROM student ORDER BY student_no, student_name;
```

上述排序可以使用未查询的列。

排序也可以使用查询列的相对位置，这种方法不推荐使用。

``` sql
SELECT student_no, student_name FROM student ORDER BY 1, 2;
```

排序默认是升序的，可以使用ASC(升序)和DESC(将序)来指定排序的方式。

``` sql
SELECT student_no, student_name FROM student ORDER BY student_no ASC, student_name DESC;
```

文本性数据的排序规则可以改变。

排序为SQL的最后一个子句。

* 过滤数据

``` sql
SELECT student_no, student_name FROM student WHERE student_no = '2019001001';
```

WHERE常用操作符（不同的DBMS支持的操作符不同）

|操作符|说明|
|:----|:----|
|=|等于|
|<>  !=|不等于|
|<|小于|
|<=|小于等于|
|!<|不小于|
|>|大于|
|>=|大于等于|
|!>|不大于|
|BETWEEN AND|在两个值之间|
|IS NULL|判断为空|
|AND|与|
|OR|或|
|IN|匹配多个值|
|NOT|否定其后面的特定操作符或关键字|

数据库中NULL表示无值，其需要单独用操作符判断，例如过滤一个列不等于3的数据，即使有该列为null的数据，其返回行也不会包含该数据。

* 通配符

``` sql
SELECT student_no, student_name FROM student WHERE student_no LIKE '2019%';
```

LIKE从技术上来说是谓词。

通配符只适用于文本字段。

%表示任意个字符，_表示单个字符。

* 字段操作

+，||，Concat可以用来拼接值，不同的DBMS支持的操作符不一样。

``` sql
SELECT CONCAT(student_no, ':', student_name) FROM student;
```

TRIM函数可以去掉字符串两边的空格，RTRIM函数可以去掉字符串右面的空格，LTRIM函数可以去掉字符串左面的空格。

AS可以用来给字段起别名。

``` sql
SELECT CONCAT(student_no, ':', student_name) AS student_info FROM student;
SELECT CONCAT(student_no, ':', student_name) AS "001" FROM student;
```

* 数值计算

查询列之间可以使用+，-，*，/来进行计算。

* 函数

不同的DBMS对函数的支持不同。

常见的字段处理函数

|处理数据类型|函数|说明|
|:----|:----|:----|
|文本类型|LENGTH()|返回字符串的长度|
|文本类型|LOWER()|将字符串转换为小写|
|文本类型|UPPER()|将字符串转换为大写|
|数值类型|ABS()|返回数值的绝对值|

常见的聚集函数

|函数|说明|
|:----|:----|
|COUNT()|返回某列的行数|
|AVG()|返回某列数值的平均值|
|SUM()|返回某列数值的和|
|MAX()|返回某列数值的最大值|
|MIN()|返回某列数值的最小值|

* 分组

GROUP BY可以用来对数据进行分组，HAVING可以用来过滤分组。

``` sql
SELECT student_grade, AVG(student_age) FROM student GROUP BY student_grade HAVING AVG(student_age) > 18;
```

* 子查询

查询结果列和查询条件中的过滤值可以用子查询SQL。

``` sql
SELECT student_no, student_name FROM student WHERE student_no IN (SELECT DISTINCT student_no FROM score WHERE score = 100);
SELECT student.student_name, (SELECT AVG(score.score) FROM score WHERE score.student_no = student.student_no) AS avg_score FROM student;
```

* 关联查询

一些DBMS会对关联表的最大数量有限制。

大多数的DBMS的关联要比子查询效率高。

student表数据

|student_no|student_name|student_age|student_sex|student_grade|
|:----|:----|:----|:----|:----|
|2019001001|小黑|18|00|1|
|2019001002|小白|20|01|1|

score表数据

|student_no|subject|score|
|:----|:----|:----|
|2019001001|语文|100|
|2019001001|数学|80|
|2019001001|英语|60|
|2019001003|语文|100|

``` sql
SELECT student.student_no, student.student_name, score.score FROM student, score WHERE student.student_no = score.student_no;
SELECT student.student_no, student.student_name, score.score FROM student INNER JOIN score ON student.student_no = score.student_no;
```

上述SQL执行结果一样。

|student_no|student_name|score|
|:----|:----|:----|
|2019001001|小黑|100|
|2019001001|小黑|80|
|2019001001|小黑|60|

第一条SQL如果没有WHERE条件的话，返回的结果是笛卡尔积，即查询条数为两个表条数的乘积。

INNER JOIN和JOIN是相同的。

``` sql
SELECT student.student_no, student.student_name, score.score FROM student LEFT JOIN score ON student.student_no = score.student_no;
```

|student_no|student_name|score|
|:----|:----|:----|
|2019001001|小黑|100|
|2019001001|小黑|80|
|2019001001|小黑|60|
|2019001002|小白|NULL|

``` sql
SELECT student.student_no, student.student_name, score.score FROM student RIGHT JOIN score ON student.student_no = score.student_no;
```

|student_no|student_name|score|
|:----|:----|:----|
|2019001001|小黑|100|
|2019001001|小黑|80|
|2019001001|小黑|60|
|NULL|NULL|100|

FULL JOIN也是存在的，但是有些DBMS不支持，MySQL就不支持。

``` sql
SELECT student_no, student_name FROM student
UNION
SELECT student_no, student_name FROM student WHERE student_no = '2019001001';
```

|student_no|student_name|
|:----|:----|
|2019001001|小黑|
|2019001002|小白|

``` sql
SELECT student_no, student_name FROM student
UNION ALL
SELECT student_no, student_name FROM student WHERE student_no = '2019001001';
```

|student_no|student_name|
|:----|:----|
|2019001001|小黑|
|2019001002|小白|
|2019001001|小黑|

UNION会去掉重复数据，UNION ALL不会去掉重复数据。

``` sql
SELECT student_no, student_name FROM student
UNION ALL
SELECT student_no, student_name FROM student WHERE student_no = '2019001001'
ORDER BY student_no;
```

|student_no|student_name|
|:----|:----|
|2019001001|小黑|
|2019001001|小黑|
|2019001002|小白|

##### 删除

``` sql
DELETE FROM student WHERE student_no = '2019001002';
```

如果没有WHERE则删除整个表的数据。

``` sql
TRUNCATE TABLE student;
```

该语句也可以删除整个表的数据，且速度更快，因为不需要记录数据的变动。

##### 修改

``` sql
UPDATE student SET student_age = 20 WHERE student_no = '2019001001';
```

如果没有WHERE则作用整个表的数据。