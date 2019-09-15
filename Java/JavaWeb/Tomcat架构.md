### 概述

Tomcat是Apache下的一个Web应用服务器，其本身是一个Servlet容器。

### Tomcat架构图

<img src="./Java/JavaWeb/image/Tomcat架构.png" alt="Tomcat架构.png"/>

### 组件

* Server

Server表示服务器，其是Tomcat中最顶层的容器，一个Tomcat只能有一个Server。

* Service

Service表示服务，一个Server可以有多个Service，每个Service有多个Connector和一个Container。

* Connector

Connector表示连接器，其处理外界的请求并封装为对象交给容器处理。

* Container

Container表示容器，其用来处理Connector传递过来的请求。

### Connector

* ProtocolHandler

Connector使用ProtocolHandler来处理请求，其包括EndPoint，Processor，Adapter。

* Endpoint

处理Socket连接。

* Processor

将Socket信息封装为Request和Response对象。

* Adapter

将Request和Response装换为ServletRequest和ServletResponse。

### Container

* Engine

Engine是容器的实现，一个Service只能有一个Engine。

* Host

Host表示虚拟主机，一个Engine可以有多个Host。

* Context

Context表示应用，一个Host可以有多个Context。

* Wrapper

Wrapper表示Servlet，其是Servlet的封装，一个Context可以有多个Wrapper。