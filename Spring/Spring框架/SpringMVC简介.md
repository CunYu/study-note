### 概述

SpringMVC是Spring中Web模块中的框架，其基于Servlet。

### 处理流程图

<img src="./Spring/Spring框架/image/SpringMVC处理流程图.png" alt="SpringMVC处理流程图"/>

### 原理

* DispatcherServlet

分发Servlet，SpringMVC中的核心，其是一个Servlet，其会拦截处理所有请求。

* HandlerMapping

其用来存储url和Controller中方法的映射关系，DispatcherServlet会根据url在HandlerMapping寻找合适的Handler.

* Handler

处理器，Controller中的方法是处理器，但处理器不仅仅只有Controller中的方法，Handler用到了适配器模式。

* ViewResolver

视图处理器，其根据Handle返回的ModelAndView来寻找相应的视图并返回。