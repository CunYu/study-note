### 服务发现组件

使用RestTemplate调用存在弊端性，其使用起来十分不灵活，当服务提供者IP或Port发现变化时，只能通过修改配置重新发布解决，其也无法动态伸缩，无法动态的增减节点。

##### 功能

* 服务注册表，服务发现组件的核心，其用来储存各个微服务的相关信息，并提供查询，注册和注销功能。

* 服务注册与发现，当微服务启动时，其会将自己的信息注册到服务发现组件中，微服务也可以通过服务发现组件中查询可用的微服务列表。

* 服务检查，微服务发现组件会通过通信（例如心跳）来检测已注册的微服务，如果在一定时间内服务发现组件没和某个已注册的微服务发生通信，那么服务发现组件会注销该微服务。

### Eureka

Eureka是Netflix开源的服务发现组件，其本身是一个基于REST的服务。

Eureka包含Server和Client两部分，Client是一个java客户端。

Eureka Client会周期性的向Eureka Server发送心跳，默认为30s。

Eureka Server在一定时间内没接收到已注册的Eureka Client的通信，那么其会注销掉该Eureka Client示例，默认时间为90s。

默认情况下，Eureka Server同时也是Eureka Client，多个Eureka Server实例会同步服务注册表。

Eureka Client会缓存服务注册表中的数据，其不需要每次都查询Eureka Server，而且自身缓存，即使Eureka Server全部宕机，Eureka Client也可以从缓存读取数据。

### 依赖包

``` groovy
compile("org.springframework.cloud:spring-cloud-starter-netflix-eureka-server")
compile("org.springframework.cloud:spring-cloud-starter-netflix-eureka-client")
```

### Eureka Server

##### 单机

* 所需依赖

``` groovy
compile("org.springframework.cloud:spring-cloud-starter-netflix-eureka-server")
```

* 启动类

``` java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

* application.yml

``` yml
spring:
  application:
    name: eurekaServer
server:
  port: 8082
eureka:
  client:
    # 是否将自己注册到Eureka Server，默认为true
    register-with-eureka: false
    # 是否从Eureka Server获取注册信息，默认为true
    fetch-registry: false
```

* 访问地址

[http://localhost:8082/](http://localhost:8082/)

##### 集群

Eureka Server通过互相注册的方式实现集群。

* 所需依赖

``` groovy
compile("org.springframework.cloud:spring-cloud-starter-netflix-eureka-server")
compile("org.springframework.cloud:spring-cloud-starter-netflix-eureka-client")
```

* 启动类

``` java
@EnableEurekaServer
@EnableEurekaClient
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

* application.yml

``` yml
spring:
  application:
    name: eurekaServer
---
spring:
  profiles: eurekaServerOne
server:
  port: 8082
eureka:
  instance:
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://root:root@localhost:8083/eureka/,http://root:root@localhost:8084/eureka/
---
spring:
  profiles: eurekaServerTwo
server:
  port: 8083
eureka:
  instance:
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://root:root@localhost:8082/eureka/,http://root:root@localhost:8084/eureka/
---
spring:
  profiles: eurekaServerThree
server:
  port: 8084
eureka:
  instance:
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://root:root@localhost:8082/eureka/,http://root:root@localhost:8083/eureka/
```

###### 说明

上面示例需以配置文件中的不同profile启动。

在Spring Cloud Edgware之前，需要手动添加@EnableEurekaClient注解，在更高的版本中，只需要添加相关的依赖，不需要自己手动添加@EnableEurekaClient注解，如不需要自动注册，需要在配置文件中声明关闭。

默认情况下，Eureka Client使用hostname进行注册，eureka.instance.prefer-ip-address为true则表示Eureka Client使用ip进行注册。

###### 安全认证

Eureka Server允许匿名登陆，这样并不安全，我们可以添加用户认证。

* 依赖包

``` groovy
compile("org.springframework.boot:spring-boot-starter-security")
```

* application.yml相关配置

``` yml
spring:
  security:
    basic:
      enabled: true
    user:
      name: root
      password: root
```

* 注册地址前面需添加root:root@

* 由于新版security默认开启了csrf校验，我们需要关闭它。

``` java
@EnableWebSecurity
public class WebSerurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        super.configure(http);
    }
}
```

###### 参考

[https://github.com/CunYu/spring-cloud-demo/tree/master/eureka-server](https://github.com/CunYu/spring-cloud-demo/tree/master/eureka-server)

### Eureka Client

* 依赖包

``` groovy
compile("org.springframework.cloud:spring-cloud-starter-netflix-eureka-client")
```

* 启动类

``` java
@EnableEurekaClient
@SpringBootApplication
public class EurekaClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}
```

* application.yml

``` yml
spring:
  application:
    name: eurekaClient
server:
  port: 8085
eureka:
  instance:
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://root:root@localhost:8082/eureka/,http://root:root@localhost:8083/eureka/,http://root:root@localhost:8084/eureka/
```

##### 参考

[https://github.com/CunYu/spring-cloud-demo/tree/master/eureka-client](https://github.com/CunYu/spring-cloud-demo/tree/master/eureka-client)

### Eureka相关

##### Eureka元数据

Eureka元数据分为两类，标准元数据和自定义元数据。

标准元数据存储在服务注册表中，自定义元数据以key-value形式存储配置文件的eureka.instance.metadata-map属性中。

DiscoveryClient中的getInstances(serviceId)方法可查询指定微服务在Eureka Server上的实例（包含元数据）列表。

##### Eureka的REST端点

Eureka Server提供了一些用以管理Eureka Server的REST端点，可以使用XML或者json与这些端点通信，默认是XML。

##### Eureka自我保护模式

一般情况下，当Eureka Server在一定时间内没有收到已注册Eureka Client的通信信息时，其会注销该实例，当发生网络故障时，会发生大规模的通信失败，但是这时候微服务本身是健康的，此时不应该注销该微服务，Eureka Server通过自我保护模式来解决这个问题。

当Eureka Server在短时间内失去大量客户端的通信信息时，其会进入自我保护模式，其会保护服务注册表中的数据，不再删除服务注册表中的任何数据。

配置文件中的eureka.server.enable-self-preservation属性为false时，禁用自我保护模式。

##### 多网卡IP选择

Spring Cloud提供了自动选择IP的功能，其可以通过配置文件来修改。

##### Eureka健康检查

Eureka可通过页面查询各微服务的状态，Eureka Server通过与Eureka Client的通信机制（心跳）来判断状态，Eureka Client也可以将自己的健康状态同步到Eureka Server。

配置文件中的eureka.client.healthcheck.enabled属性为true时，Eureka Client会将自己的健康状态同步到Eureka Server，该属性只能配置在application文件中。

##### 去除Jersey依赖

默认情况下Eureka Client通过Jersey与Eureka Server交互。

在Spring Cloud Edgware中，Spring Client支持使用其他客户端与Eureka Server通信，其可通过去除Jersey依赖包的方法来禁用Jersey，去除Jersey依赖后，Spring Cloud会自动配置一个基于RestTemplate的HTTP客户端。