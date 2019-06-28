本文介绍了Feign相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 概述

Feign是Netflix开发的声明式，模板化的HTTP客户端。

Spring Cloud对Feign进行了增强，其时Feign支持Spring MVC注解，并整合了Ribbon和Eureka。

### Spring Cloud整合Eureka，Ribbon，Feign

##### Feign Spring Cloud依赖包

``` groovy
compile("org.springframework.cloud:spring-cloud-starter-openfeign")
```

##### 启动类

``` java
@EnableFeignClients
@SpringBootApplication
public class EurekaRibbonFeignConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaRibbonFeignConsumerApplication.class, args);
    }
}
```

##### 服务提供者Controller

``` java
@RestController
@RequestMapping("calculation")
public class CalculationController {

    @GetMapping("add")
    public CalculationResult add(@RequestParam("paramOne") BigDecimal paramOne, @RequestParam("paramTwo") BigDecimal paramTwo) {

        CalculationResult calculationResult = new CalculationResult();
        calculationResult.setParamOne(paramOne);
        calculationResult.setParamTwo(paramTwo);
        calculationResult.setResult(paramOne.add(paramTwo));
        return calculationResult;
    }
}
```

##### 服务消费者Feign调用

``` java
@FeignClient(name = "eurekaProvider")
@RequestMapping("calculation")
public interface EurekaProviderFeignClient {

    @GetMapping("add")
    CalculationResult add(@RequestParam("paramOne") BigDecimal paramOne, @RequestParam("paramTwo") BigDecimal paramTwo);
}
```

@FeignClient(name = "eurekaProvider")注解中的eurekaProvider是用来创建Ribbon负载均衡器的，Ribbon会去Eureka中查询名为eurekaProvider的服务实例列表，然后根据规则选择一个合适的实例。

``` java
@Slf4j
@RestController
@RequestMapping("display")
public class DisplayController {

    @Autowired
    private EurekaProviderFeignClient eurekaProviderFeignClient;

    @GetMapping("result")
    public String result(@RequestParam("paramOne") BigDecimal paramOne, @RequestParam("paramTwo") BigDecimal paramTwo) {

        CalculationResult calculationResult = eurekaProviderFeignClient.add(paramOne, paramTwo);
        return calculationResult.getParamOne() + " + " + calculationResult.getParamTwo() + " = " + calculationResult.getResult();
    }
}
```

Feign支持继承，所以这里可以让服务提供者提供接口，服务提供者Controller实现接口，服务消费者继承接口，进而极大的方便了服务消费者Feign Client配置。

##### Feign配置

* java配置

可通过如下为feign指定单独的配置文件。

``` java
@FeignClient(name = "eurekaProvider", configuration = Xxx.class)
```

Feign配置类不应该被启动类的@ComponentScan扫描到，如果被扫描到，该配置类会被所有Feign Client共享。

* 属性配置

配置单独的Feign客户端

``` yml
feign:
  client:
    config:
      eurekaProvider:
        # 建立连接超时时间
        connectTimeout: 5000
        # 等待目标服务器响应时间
        readTimeout: 5000
```

配置所有的Feign客户端

``` yml
feign:
  client:
    config:
      default:
        # 建立连接超时时间
        connectTimeout: 5000
        # 等待目标服务器响应时间
        readTimeout: 5000
```

属性配置的优先级高于java配置，而且使用起来相对方便，建议使用属性配置。

##### 参考

[https://github.com/CunYu/spring-cloud-demo/tree/master/eureka-ribbon-feign-consumer](https://github.com/CunYu/spring-cloud-demo/tree/master/eureka-ribbon-feign-consumer)

### 手动创建Feign

当自定义Feign不能满足需求时，我们可以通过Feign Builder API手动创建API。

### Feign压缩

Feign支持对请求和响应进行压缩。

``` yml
feign:
  compression:
    request:
      enabled: true
      mime-types: application/json,text/xml
      # 压缩的最小阀值 默认为2048
      min-request-size: 2048
```

### Feign日志级别

Feign可以通过配置指定Feign Client的日志级别。

|日志级别|说明|
|:----|:----|
|NONE|不打印任何日志（默认）|
|BASIC|仅打印请求方法，URL，响应状态编码和执行时间|
|HEADERS|在BASIC的基础上，增加请求和响应的Header|
|FULL|打印请求和响应的header，body和元数据|

### Feign文件处理

Feign通过子项目Feign-Form来实现文件上传和下载。