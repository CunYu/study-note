### 概述

Dubbo是一个开源的Java RPC框架，其由阿里开发，现归属于Apache。

### 结构

<img src="./框架工具/Dubbo/image/Dubbo结构.png" alt="Dubbo结构"/>

* Provider

提供服务的应用，其需要运行在容器中，容器可以为Spring，Jetty，Log4j容器。

* Consumer

消费服务的应用。

* Registry

注册中心，服务发现组件。

* Monitor

监控中心，其可以监控服务的调用次数，调用时间。

### 通信

Dubbo通信基于Netty，也可以选择其他的通信框架。

Provider，Comsumer，Regitry之间使用长连接通信。

Comsumer本地会存在缓存，其会缓存Provider相关信息。

Comsumer启动时会检查服务提供者是否可用，可用check="false"关闭检查。

Comsumer可以指定服务提供者。

### Dubbo配置优先级

Comsumer方法配置 > Provider方法配置 > Consumer接口配置 > Provider接口配置 > Consumer全局配置 > Provider全局配置

##### Dubbo常见支持协议

* Dubbo（默认）

* HTTP

* rmi

* hessian

* redis

### 序列化

Dubbo默认使用Hessian序列化，也可以使用FastJson，Java自带序列化。

### 集群容错

* Failover

当调用失败时，重试其他服务器，可通过retries配置重试的次数。集群容错默认值。

* Failfast

快速失败，一次调用，失败立即报错。

* Failsafe

失败安全，忽略失败。

* Failback

失败恢复，后台会定时重发。

* Forking

并行调用服务提供者，有一个成功即返回，forks可以配置并行数。

* Broadcast

依次调用服务提供者，有一个失败即失败。