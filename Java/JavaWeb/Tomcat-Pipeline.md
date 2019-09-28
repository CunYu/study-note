### Container

Container是由Engine，Host，Context，Wrapper组成。

Engine，Host，Context，Wrapper都是Container接口的实现，Tomcat使用Pipeline来处理Container接收到的请求。

Tomcat-Container常见类结构

<img src="./Java/JavaWeb/image/Tomcat-Container常见类结构.png" alt="Tomcat-Container常见类结构"/>

### Pipeline

Pipeline是由Valve组成的，每个Pipeline可以有零个或多个普通Valve和一个基础Valve组成。

Tomcat-Pipeline和Valve常见类结构

<img src="./Java/JavaWeb/image/Tomcat-Pipeline和Valve常见类结构.png" alt="Tomcat-Pipeline和Valve常见类结构"/>

### Pipeline处理请求流程

Pipeline中的Value实际上运用了责任连模式，每个Value会存储下一个Value，其通过invoke方法来执行相关处理。

<img src="./Java/JavaWeb/image/Tomcat-Pipeline处理请求流程.png" alt="Tomcat-Pipeline处理请求流程"/>