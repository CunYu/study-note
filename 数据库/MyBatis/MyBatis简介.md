### 概述

MyBatis是Apache的一个持久层框架，其对JDBC进行了封装，简化了数据库操作，其曾用名是iBatis。

Mybatis将SQL结果映射为Java对象。

### SqlSessionFactory

SqlSessionFactory是MyBatis应用的核心，其用来生产SqlSession。

SqlSessionFactory通过SqlSessionFactoryBuilder构建，SqlSessionFactoryBuilder可以通过XML配置文件或者Configuration配置类构建。

SqlSession是与数据库交流的Sql会话，其不是线程安全的，其由SqlSessionFactory生产。

### Mapper映射

MyBatis需要声明Mapper，Mapper代表的是具体的SQL及SQL参数，结果集类型，Mybatis需要手工配置映射关系。

### MyBatis工作原理

* 读取配置文件，mybatis-config.xml为全局配置文件，其可以配置数据库，事务，Mapper等信息。

* 根据配置信息生成SqlSessionFactoryBuilder

* 通过SqlSessionFactoryBuilder创建SqlSessionFactory

* SqlSessionFactory可以用来创建SqlSession

* 应用调用Mapper，根据Mapper找到相应的SQL，以及解析参数

* 通过SqlSession与数据库通信后，将返回的结果封装为Java对象返回

### MyBatis缓存

MyBatis缓存会对查询结果进行缓存。

* 一级缓存

同一个SqlSession中，完全相同的SQL只会执行一次，第一次执行会去数据库执行得到结果，该结果会被SqlSession缓存下来，第二此执行完全相同的SQL时，会从缓存中直接读取（缓存未失效）。

如果SqlSession中执行了修改等操作，会使其缓存失效。

使用一级缓存时，会发生脏数据的情况，在一个SqlSession中执行SQL，其将结果缓存下来，另一个SqlSession修改了数据，但是已缓存结果的那个SqlSession执行SQL还是会从缓存中拿数据。

可以在xml配置文件中给相应的SQL配置flushCache="true"来清除SqlSession中的所有缓存。

* 二级缓存

一级缓存基于SqlSession，SqlSession之间无法共享。

二级缓存基于namespace，同一个SqlSessionFactory创建的SqlSession可以共享。

一级缓存，二级缓存都存在时，优先使用二级缓存。

二级缓存需要执行SQL的SqlSession提交事务才生效。

二级缓存在多表关联查时存在脏数据的问题，而多表关联查SQL所在的namespace无法发现其他namespace中对表的修改，所以在其他namespace中修改数据时无法导致缓存失效，可以通过引入其他namespace来解决这个问题。