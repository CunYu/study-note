### Session

* servlet-demo项目结构

``` text
src
--main
----java
------DemoServlet
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

        HttpSession httpSession = req.getSession();
        String hello = (String) httpSession.getAttribute("hello");

        if (null == hello) {
            httpSession.setAttribute("hello", "Hello world!");
        } else {
            PrintWriter printWriter = resp.getWriter();
            printWriter.write(hello);
        }
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
```

通过tomcat启动，浏览器访问 http://localhost:8080/servlet-demo/demo 两次，第二次显示为空，第二次显示。

``` text
Hello world!
```

### Cookie

* servlet-demo项目结构

``` text
src
--main
----java
------DemoServlet
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

        Cookie[] cookies = req.getCookies();

        if (null == cookies) {

            Cookie cookie = new Cookie("hello", "Hello world!");
            resp.addCookie(cookie);
        } else {

            for (Cookie cookie : cookies) {
                if ("hello".equals(cookie.getName())) {
                    PrintWriter printWriter = resp.getWriter();
                    printWriter.write(cookie.getValue());
                    return;
                }
            }

            Cookie cookie = new Cookie("hello", "Hello world!");
            resp.addCookie(cookie);
        }
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
```

通过tomcat启动，浏览器访问 http://localhost:8080/servlet-demo/demo 两次，第二次显示为空，第二次显示。

``` text
Hello world!
```