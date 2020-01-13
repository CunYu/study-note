### 概述

Spring使用Resource来对资源进行封装。

### 常见Resource

|名称|说明|
|:----|:----|
|InputStreamResource|输入流资源|
|ByteArrayResource|字节数组资源|
|ServletContextResource|ServletContext资源|
|ClassPathResource|类路径资源|
|FileSystemResource|文件系统资源|
|UrlResource|Url资源|

### ResourceLoader

Spring为了简化Resource的使用，其提供了ResourceLoader来方便Resource的获取。

``` java
ResourceLoader resourceLoader = new DefaultResourceLoader();
Resource resource = resourceLoader.getResource("classpath:xxx");
```

ApplicationContext实现了ResourceLoader。

* classpath:xxx

表示获取ClassPathResource资源。

* http:xxx/file:xxx

表示获取UrlResource资源。