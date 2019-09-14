### 事务概述

事务是对数据库数据操作的一个基本单位，其包含一条或多条数据库数据操作。

事务遵循ACID特性。

* Atomicity（原子性）

事务包含的数据库数据操作要不全部操作成功，要不全部操作不成功。

* Consistency（一致性）

事务使数据库数据从一个一致性状态变更为另一个一致性状态，一致性状态是根据具体的情况具体定义的。

* Isolation（隔离性）

并发操作数据库数据时，各个事务之间是互不干扰，互相隔离的。

* Durability（持久性）

事务完成后，对数据库的数据来说是永久变化的。

### 事务相关概念简述

* 脏读

在一个事务处理过程中读取到了另一个事务未提交的数据。

* 不可重复读

在一个事务处理过程中，对同一数据的多次查询，结果不相同。针对单一数据，主要为update。

* 虚读（幻读）

在一个事务处理过程中，多次查询的结果集不一样。针对结果集数量，主要为insert，delete。

### Spring 事务主要API

* PlatformTransactionManager

事务管理平台，是Spring事务的核心接口。

* TransactionDefinition

事务定义，该接口定义了Spring遵循的事务属性。

* TransactionStatus

事务状态，事务的相关状态展示。

### Spring 事务传播行为

|名称|说明|
|:----|:----|
|PROPAGATION_REQUIRED|如果当前有事务就加入当前事务，如果当前没有事务则新建一个事务。默认事务传播行为。|
|PROPAGATION_SUPPORTS|如果当前有事务就加入当前事务，如果当前没有事务则以无事务的方式执行。|
|PROPAGATION_MANDATORY|如果当前有事务就加入当前事务，如果当前没有事务则抛出异常。|
|PROPAGATION_REQUIRES_NEW|创建一个新事务，如果当前有事务则将当前事务挂起。新事务执行完后，当前事务才恢复执行。|
|PROPAGATION_NOT_SUPPORTED|以非事务方式执行，如果当前有事务就挂起当前事务，直至执行完毕后，当前事务恢复执行。|
|PROPAGATION_NEVER|以非事务方式执行，如果当前有事务则抛出异常。|
|PROPAGATION_NESTED|如果当前有事务就以嵌套事务的方式加入当前事务，如果当前没有事务则相当于PROPAGATION_REQUIRED。|

PS

嵌套事务：嵌套事务依赖于外部事务，外部事务提交，嵌套事务提交，外部事务回滚，嵌套事务回滚，嵌套事务回滚不会引起外部事务回滚。

### Spring 事务隔离级别

|名称|说明|
|:----|:----|
|ISOLATION_DEFAULT|使用底层数据库默认隔离级别。|
|ISOLATION_READ_UNCOMMITTED|最低的隔离级别，脏读，不可重复读，虚读（幻读）可以发生。|
|ISOLATION_READ_COMMITTED|该级别避免了脏读的发生，不可重复读，虚读（幻读）可以发生。|
|ISOLATION_REPEATABLE_READ|该级别避免了脏读，不可重复读的发生，虚读（幻读）可以发生。|
|ISOLATION_SERIALIZABLE|最高的隔离级别，也是消耗最大的隔离级别，该级别避免了禁止了脏读，不可重复读，虚读（幻读）的发生。|

### 事务超时

执行事务超过指定的时间后，会抛出异常。

### 只读事务

只有读操作的事务，可以设置为只读事务，底层会有可能针对该类事务进行特定的优化，这样会有可能提升一定的性能。

### Spring 事务实现方式

##### 编程式事务

###### 使用TransactionTemplate

主要代码展示：

依赖包

``` groovy
compile('org.springframework.boot:spring-boot-starter-web')
compile('org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.2')
compile('com.alibaba:druid:1.1.10')
compile('mysql:mysql-connector-java:8.0.12')
```

application.yml

``` yml
spring:
    datasource:
        url: jdbc:mysql://192.168.99.100:3306/demo?useSSL=false&allowPublicKeyRetrieval=true
        username: root
        password: 123456
        driverClassName: com.mysql.cj.jdbc.Driver
        initialSize: 5
        minIdle: 5
        maxActive: 100

mybatis:
    mapper-locations: classpath:/mapper/*.xml

transactionTemplate:
    propagationBehaviorName: PROPAGATION_REQUIRED
    isolationLevelName: ISOLATION_SERIALIZABLE
    timeout: 60
```

applicationContext.xml

``` xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="url" value="${spring.datasource.url}"/>
    <property name="username" value="${spring.datasource.username}"/>
    <property name="password" value="${spring.datasource.password}"/>
    <property name="driverClassName" value="${spring.datasource.driverClassName}"/>
    <property name="initialSize" value="${spring.datasource.initialSize}"/>
    <property name="minIdle" value="${spring.datasource.minIdle}"/>
    <property name="maxActive" value="${spring.datasource.maxActive}"/>
</bean>

<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="propagationBehaviorName" value="${transactionTemplate.propagationBehaviorName}"/>
    <property name="isolationLevelName" value="${transactionTemplate.isolationLevelName}"/>
    <property name="timeout" value="${transactionTemplate.timeout}"/>
</bean>
```

springboot启动类

``` java
@ImportResource("classpath:applicationContext.xml")
@MapperScan("xxx.xxx.xxxxx.mapper")
```

使用事务代码

``` java
@Autowired
private TransactionTemplate transactionTemplate;

public void tracsactionTest() {

    // 无返回值
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
        @Override
        protected void doInTransactionWithoutResult(TransactionStatus status) {
            // 事务代码
            // status.setRollbackOnly() 手动回滚，发生异常时会自动回滚，一般不需要需要。
        }
    });

    // 有返回值
    int num = transactionTemplate.execute(new TransactionCallback<Integer>() {
        @Override
        public Integer doInTransaction(TransactionStatus status) {
            // 事务代码
            // status.setRollbackOnly() 手动回滚，发生异常时会自动回滚，一般不需要需要。
            return 1;
        }
    });
}
```

###### 使用PlatformTransactionManager

需要手动提交，回滚。

主要代码展示：

依赖包

``` groovy
compile('org.springframework.boot:spring-boot-starter-web')
compile('org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.2')
compile('com.alibaba:druid:1.1.10')
compile('mysql:mysql-connector-java:8.0.12')
```

application.yml

``` yml
spring:
    datasource:
        url: jdbc:mysql://192.168.99.100:3306/demo?useSSL=false&allowPublicKeyRetrieval=true
        username: root
        password: 123456
        driverClassName: com.mysql.cj.jdbc.Driver
        initialSize: 5
        minIdle: 5
        maxActive: 100

mybatis:
    mapper-locations: classpath:/mapper/*.xml
```

applicationContext.xml

``` xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="url" value="${spring.datasource.url}"/>
    <property name="username" value="${spring.datasource.username}"/>
    <property name="password" value="${spring.datasource.password}"/>
    <property name="driverClassName" value="${spring.datasource.driverClassName}"/>
    <property name="initialSize" value="${spring.datasource.initialSize}"/>
    <property name="minIdle" value="${spring.datasource.minIdle}"/>
    <property name="maxActive" value="${spring.datasource.maxActive}"/>
</bean>
```

springboot启动类

``` java
@ImportResource("classpath:applicationContext.xml")
@MapperScan("xxx.xxx.xxxxx.mapper")
```

使用事务代码

``` java
@Autowired
private PlatformTransactionManager platformTransactionManager;

public void tracsactionTest() {

    DefaultTransactionDefinition defaultTransactionDefinition = new DefaultTransactionDefinition();
    defaultTransactionDefinition.setPropagationBehaviorName("PROPAGATION_REQUIRED");
    defaultTransactionDefinition.setIsolationLevelName("ISOLATION_SERIALIZABLE");
    defaultTransactionDefinition.setTimeout(60);

    TransactionStatus transactionStatus = platformTransactionManager.getTransaction(defaultTransactionDefinition);
    try {
        // 事务代码
        platformTransactionManager.commit(transactionStatus);
    } catch (Exception e) {
        platformTransactionManager.rollback(transactionStatus);
    }
}
```

##### 声明式事务

###### XML配置

主要代码展示：

依赖包

``` groovy
compile('org.springframework.boot:spring-boot-starter-web')
compile('org.springframework.boot:spring-boot-starter-aop')
compile('org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.2')
compile('com.alibaba:druid:1.1.10')
compile('mysql:mysql-connector-java:8.0.12')
```

application.yml

``` yml
spring:
    datasource:
        url: jdbc:mysql://192.168.99.100:3306/demo?useSSL=false&allowPublicKeyRetrieval=true
        username: root
        password: 123456
        driverClassName: com.mysql.cj.jdbc.Driver
        initialSize: 5
        minIdle: 5
        maxActive: 100

mybatis:
    mapper-locations: classpath:/mapper/*.xml
```

applicationContext.xml

``` xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="url" value="${spring.datasource.url}"/>
    <property name="username" value="${spring.datasource.username}"/>
    <property name="password" value="${spring.datasource.password}"/>
    <property name="driverClassName" value="${spring.datasource.driverClassName}"/>
    <property name="initialSize" value="${spring.datasource.initialSize}"/>
    <property name="minIdle" value="${spring.datasource.minIdle}"/>
    <property name="maxActive" value="${spring.datasource.maxActive}"/>
</bean>

<tx:advice id="transactionInterceptor" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="insert*" propagation="REQUIRED" isolation="SERIALIZABLE" timeout="60"/>
        <!--可配置多个tx:method-->
    </tx:attributes>
</tx:advice>

<aop:config>
    <aop:pointcut id="transactionPointcut" expression="execution(* xxx.xxx.xxxxx.service..*.*(..))"/>
    <aop:advisor advice-ref="transactionInterceptor" pointcut-ref="transactionPointcut"/>
</aop:config>
```

springboot启动类

``` java
@ImportResource("classpath:applicationContext.xml")
@MapperScan("xxx.xxx.xxxxx.mapper")
```

###### 注解

主要代码展示：

依赖包

``` groovy
compile('org.springframework.boot:spring-boot-starter-web')
compile('org.springframework.boot:spring-boot-starter-aop')
compile('org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.2')
compile('com.alibaba:druid:1.1.10')
compile('mysql:mysql-connector-java:8.0.12')
```

application.yml

``` yml
spring:
    datasource:
        url: jdbc:mysql://192.168.99.100:3306/demo?useSSL=false&allowPublicKeyRetrieval=true
        username: root
        password: 123456
        driverClassName: com.mysql.cj.jdbc.Driver
        initialSize: 5
        minIdle: 5
        maxActive: 100

mybatis:
    mapper-locations: classpath:/mapper/*.xml
```

applicationContext.xml

``` xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="url" value="${spring.datasource.url}"/>
    <property name="username" value="${spring.datasource.username}"/>
    <property name="password" value="${spring.datasource.password}"/>
    <property name="driverClassName" value="${spring.datasource.driverClassName}"/>
    <property name="initialSize" value="${spring.datasource.initialSize}"/>
    <property name="minIdle" value="${spring.datasource.minIdle}"/>
    <property name="maxActive" value="${spring.datasource.maxActive}"/>
</bean>

<tx:annotation-driven transaction-manager="transactionManager"/>
```

springboot启动类

``` java
@ImportResource("classpath:applicationContext.xml")
@MapperScan("xxx.xxx.xxxxx.mapper")
```

事务注解

``` java
@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.SERIALIZABLE, timeout = 60)
```