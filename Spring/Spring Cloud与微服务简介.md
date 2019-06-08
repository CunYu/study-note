本文介绍了Spring Cloud与微服务相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 微服务概述

单体应用，一个归档包包含所有功能的程序。很多项目都是从单体应用开始的，但是随着发展，单体应用就变得不是那么适用了，单体应用复杂度高，可靠性低，扩展能力有限。

微服务是为了解决单体应用的弊端而诞生的一种架构模式，其将单一的应用变为一组微型服务，每个服务独自运行并通过轻量级通信机制（跨语言，跨平台）来通信。

微服务通过对单一应用进行拆分，各服务独立部署，使得每个服务的代码不会很多，降低复杂度，而且各个服务独立运行提高了服务的可靠性，避免了一处错误导致整个系统奔溃的情况，也方便系统的开发和维护，可以单独部署某一模块，也可以针对某一模块进行资源的伸缩管理，而且各个服务独立部署，相互之间通过轻量级通信机制通信，极大的提高了系统的扩展能力，技术选型也相对灵活。

微服务对运维的要求较高，分布式固有的复杂性，接口维护通信成本高。

### Spring Cloud

Spring Cloud是微服务开发框架之一。

##### Spring Cloud特点

* 约定大约配置

* 轻量级组件

* 生态丰富，功能齐全

* 灵活，可自由选择组件

##### Spring Cloud版本

Spring Cloud是以英文名+标识字母+数字的格式命名的。

英文名字的首字母是按照字母顺序来的，代表主版本的演进。

标识字母SR代表Service Release，一般是在发布内容累积到一个临界点或Bug修复后发布。SR版本之前会发布一个Release版本。

标识字母M代表MileStone，表示是里程碑版本，

数字标识发布的先后顺序。

SNAPSHOT表示快照版本，随时都有可能修改。

PRE表示预览版。

GA（(GenerallyAvailable）表示稳定版本。

##### 版本兼容性

Spring Cloud建立在Spring Boot的基础上，每个Spring Cloud版本基于Spring Boot的版本不一样。

每个Spring Cloud版本对应的组件版本也不一致，具体的版本对应可以在Spring官网查看。

### Spring Cloud搭建

##### Spring Initializr

* [https://start.spring.io/](https://start.spring.io/)

* Spring Tool Suite

* Spring Boot CLI

##### 手动搭建

可参考[https://github.com/CunYu/template/tree/master/spring-cloud-template](https://github.com/CunYu/template/tree/master/spring-cloud-template)

##### Spring Cloud RestTemplate Demo

[https://github.com/CunYu/spring-cloud-demo](https://github.com/CunYu/spring-cloud-demo)中的rest-template-provider和rest-template-consumer子项目。