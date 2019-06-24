本文介绍了Ribbon与负载均衡器相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 负载均衡器

在实际使用中，每个服务往往会部署多个实例，负载均衡器可以使服务消费者的请求按照一定的规则分摊到各个服务实例上。

### Ribbon

Ribbon是Netflix发布的负载均衡器，其可以基于负载均衡算法在服务实例中选择一个合适的服务实例。

在Spring Cloud中，Ribbon和Eureka配置使用时，Ribbon会自动从Eureka中获取服务实例列表，然后基于负载均衡算法选择一个服务实例。

### Spring Cloud整合Eureka,Ribbon

##### Ribbon Spring Cloud依赖包

``` groovy
compile("org.springframework.cloud:spring-cloud-starter-netflix-ribbon")
```

##### RestTemplate配置Ribbon负载均衡

``` java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

@LoadBalanced可以使RestTemplate整合Ribbon。

##### 使用

``` java
@Slf4j
@RestController
@RequestMapping("display")
public class DisplayController {

    @Autowired
    private RestTemplate restTemplate;
    @Autowired
    private LoadBalancerClient loadBalancerClient;

    @GetMapping("result")
    public String result(@RequestParam("paramOne") BigDecimal paramOne, @RequestParam("paramTwo") BigDecimal paramTwo) {

        CalculationResult calculationResult = restTemplate.getForObject("http://eurekaProvider/calculation/add?paramOne=" + paramOne + "&paramTwo=" + paramTwo, CalculationResult.class);
        return calculationResult.getParamOne() + " + " + calculationResult.getParamTwo() + " = " + calculationResult.getResult();
    }

    @GetMapping("choose")
    public String choose() {

        ServiceInstance serviceInstance = loadBalancerClient.choose("eurekaProvider");
        return serviceInstance.getServiceId() + ":" + serviceInstance.getHost() + ":" + serviceInstance.getPort();
    }
}
```

Ribbon和Eureka整合使用时，我们可以将IP地址替换为Eureka的虚拟主机名，Ribbon会根据虚拟主机名去Eureka中查找实例列表，并根据负载均衡规则选取一个合适的实例。

LoadBalancerClient的choose方法可以更直接的查看Ribbon所选择的服务实例，但是其不能和用@LoadBalanced修饰过的RestTemplate方法放到一起，两者会发生冲突，因为此时的RestTemplate实例是一个Ribbon客户端，其本身已包含了“choose”的行为。

虚拟主机名中不能包含'_'之类的字符。

##### Ribbon配置

* java示例

``` java
@Configuration
public class EurekaProviderRibbonConfig {

    @Bean
    public IRule eurekaProviderRibbonRule() {
        // 随机负载均衡规则
        return new RandomRule();
    }
}
```

``` java
@Configuration
@RibbonClient(name = "eurekaProvider", configuration = EurekaProviderRibbonConfig.class)
public class EurekaProviderRibbonClient {
}
```

EurekaProviderRibbonConfig配置类不应该被启动类的@ComponentScan扫描到，如果被扫描到，该配置类会被所有Ribbon Client共享。

@RibbonClients(defaultConfiguration = EurekaRibbonClientConfiguration.class)为所有Ribbon Client提供默认配置。

* 属性

``` yml
eurekaProvider:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

为所有Ribbon Client添加配置

``` yml
ribbon:
  NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

### 脱离Eureka使用Ribbon

Ribbon可以单独使用，不依赖Eureka，也可以不在Spring Cloud中使用。

### 懒加载

Spring Cloud为每个Ribbon Client维护一个应用程序上下文，其是懒加载的，在第一次请求时，才会进行加载上下文，所以首次请求往往耗时较长，我们可以配置是否懒加载。

``` yml
ribbon:
  eager-load:
    enabled: true
    clients: eurekaProvider
```

多个client可用,号分开。