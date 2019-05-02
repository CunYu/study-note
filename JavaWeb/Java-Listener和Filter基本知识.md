本文介绍了Java Listener和Filter基本知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 概述

Listener和Filter是Servlet应用中重要的组件，其和Servlet一样，可以配置在web.xml中。

其对应的加载顺序为：Listener > Filter > Servlet

### Listener

Listener是用来监听Servlet应用中的各种事件的，其运用了观察者模式，各种类型的Listener都实现了EventListener接口。

##### 常见Listener类型

* ServletContextListener（ServletContext的创建与销毁）

* ServletContextAttributeListener（ServletContext属性变化）

* ServletRequestListener（ServletRequest的创建与销毁）

* ServletRequestAttributeListener（ServletRequest属性变化）

* HttpSessionListener（HttpSession的创建与销毁）

* HttpSessionAttributeListener（HttpSession属性变化）

### Filter

Filter可以用来过滤ServletRequest和ServletResponse，其运用了责任链模式，各种类型的Filter都实现了Filter接口。

##### Filter顺序

满足过滤路径条件的Filter会根据web.xml中配置的先后顺序来执行。

### Filter接口方法

* init

* doFilter

``` java
// 保证Filter链执行下去
chain.doFilter(request, response);
```

* destroy

### Listener Filter Demo

* servlet-demo项目结构

``` text
src
--main
----java
------DemoServlet
------EncodeFilter
------RequestCountListener
----webapp
------WEB-INF
--------web.xml
build.gradle
```

* DemoServlet

``` java
public class DemoServlet extends HttpServlet {

    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        int count = (Integer) req.getServletContext().getAttribute("count");
        PrintWriter printWriter = resp.getWriter();
        printWriter.print("第" + count + "次访问！");
    }
}
```

* EncodeFilter

``` java
public class EncodeFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        response.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");

        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {

    }
}
```

* RequestCountListener

``` java
public class RequestCountListener implements ServletRequestListener {

    private int count;

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        ServletContext servletContext = sre.getServletContext();
        servletContext.setAttribute("count", ++count);
    }

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
    }
}
```

* web.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <listener>
        <display-name>requestCountListener</display-name>
        <listener-class>RequestCountListener</listener-class>
    </listener>

    <filter>
        <filter-name>encodeFilter</filter-name>
        <filter-class>EncodeFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>encodeFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <servlet>
        <servlet-name>demoServlet</servlet-name>
        <servlet-class>DemoServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>demoServlet</servlet-name>
        <url-pattern>/demo</url-pattern>
    </servlet-mapping>
</web-app>
```

* build.gradle

``` groovy
apply plugin: 'java'
apply plugin: 'war'

repositories {
    mavenCentral()
}

dependencies {
    compile("javax.servlet:javax.servlet-api:4.0.1")
}

tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
}
```

通过tomcat启动，访问http://localhost:8080/servlet-demo/demo

* 网页显示

``` text
第1次访问！
```