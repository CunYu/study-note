### 概述

MVCC的英文名为Multi-Version Concurrency Control，也就是多版本并发访问控制。

MVCC在隔离级别为读已提交，可重复读时使用。

其保证了读写不冲突，可以并发的处理数据。

### 表相关隐藏字段

* DB_TRX_ID

存储操作数据的事务ID，每次开启事务时都会生成一个有序的事务ID。

* DB_ROLL_PTR

存储上个版本数据在Undo Log中的指针，Undo Log是用来存储数据变动记录的，也就是存储历史版本数据。

### Read View

在隔离级别为读已提交时，事务中的每条SELECT语句都会生成一个Read View。

在隔离级别为可重复读时，事务中只有第一条SELECT语句会生成一个Read View，其他的SELECT语句复用第一条语句生成的Read View。

##### Read View结构

* trx_ids

当前系统活跃事务的ID集合。

* low_limit_id

当前系统活跃事务的最大ID。

* up_limit_id

当前系统活跃事务的最小ID。

* creator_trx_id

每次开启事务时都会生成一个有序的事务ID和一个Read View，该属性存储创建其的事务ID。

##### 读取方式

* 快照读

读取Undo Log中历史版本数据。

* 当前读

读取当前数据，也就是最新的数据。

##### 读取规则

数据中DB_TRX_ID小于up_limit_id当前读。

数据中DB_TRX_ID大于low_limit_id快照读。

数据中DB_TRX_ID在up_limit_id和low_limit_id之间的，如果DB_TRX_ID不在trx_ids中或者DB_TRX_ID等于creator_trx_id则为当前读，其他为快照读。

快照读需要去Undo Log中寻找适合的历史版本数据来读取。