### 面向切面编程

在OOP中，是纵向对对象进行管理，其无法对对象进行横向管理，而AOP是面向切面编程，其利用切面对对象进行了横向管理，是对OOP的补充和完善。

例如我们有个灯对象，其有发光的方法，有个书对象，其有记录的方法，如果我们想在灯每次发光开始和结束的时候进行记录，这时候OOP就不好实现这个功能，其需要将书对象引入到灯对象中，但是这样是不太合适的，灯不应该依赖书的，如果灯自己定义记录方法就更不合适了。这时候利用AOP就可以很好的满足我们的需求。

### AOP相关概念

##### 通知（Advice）

定义了横切行为。

* 前置通知（Before）

* 后置通知（After）

* 返回通知（After-returning）

* 异常通知（After-throwing）

* 环绕通知（Around）

后置通知不管通知的方法是否执行成功，只要方法完成即调用通知。

返回通知是在通知的方法执行成功后才调用通知。

##### 连接点（Join Point）

程序执行过程中可以应用通知的所有点。

##### 切点（Pointcut）

应用通知的连接点。

##### 切面（Aspect）

切面是通知和切点的结合，其定义横切行为及行为执行的位置。

##### 织入（Warving）

织入是指切面应用到目标对象并创建新的代理对象的过程。

* 编译期

* 类加载期

* 运行期

### Spring AOP管理

Spring AOP中代理对象由Spring容器统一管理。

Spring AOP中使用动态代理和cglib代理分别代理接口和类，cglib是通过继承目标类的方式来创建代理，所以其不能代理final修饰的目标类。

### Spring AOP和AspectJ区别

Spring AOP只能对方法进行切面，也就是Spring AOP的连接点都是方法。AspectJ即可以对方法进行切面，也可以对构造器和属性进行切面。

Spring AOP是通过代理对目标对象方法增强的方式实现的，其发生在运行期间。

AspectJ是通过操作字节码来实现的，其发生在编译期间。

AspectJ的性能要比Spring AOP高。

### Spring AOP实现

##### 依赖

``` groovy
compile('org.springframework.boot:spring-boot-starter-aop')
```

##### 切点表达式

``` text
execution(* xxx.Xxx.xxx(..))
```

* 表达式第一个代表方法返回类型，*代表我们不在意方法的返回类型。

* 后面的xxx.Xxx.xxx对应包路径下面的类的方法。

* ()里面代表方法的参数，（..）表示方法参数随意，也就是匹配该类所有方法名符合的方法。

``` text
within(xxx.*)
```

* 该表达式的精度为类，表示需在参数所指定的类中。

``` text
bean('xxx')
```

* 该表达式需要beanId作为参数，限定切点为指定的bean.


切点表达式支持&&，||，！符号，也可以使用and，or，not来替代，注意在XML文件中，&有特殊含义。


* *代表通配符

* 包路径中的..代表该包及其包中的所有子包。例如execution(* xxx..Xxx.*(..))。

##### 注解实现场景一

对灯每次发光进行记录。

###### Lamp接口及其实现类

``` java
public interface Lamp {

    void shine();
}
```

``` java
public class RedLamp implements Lamp {
	
    @Override
    public void shine() {
        System.out.println("发出红光");
    }
}
```

###### 定义切面

``` java
@Aspect
public class Book {

    @Before("execution(* demo.Lamp.shine(..))")
    public void startRecord() {
        System.out.println("开始记录");
    }

    @After("execution(* demo.Lamp.shine(..))")
    public void endRecord() {
        System.out.println("记录完成");
    }
}
```

###### 开启切面

如不开启切面，Book就是一个普通的bean。

``` java
@Configuration
@EnableAspectJAutoProxy
public class Config {

    @Bean
    public RedLamp redLamp() {
        return new RedLamp();
    }

    @Bean
    public Book book() {
        return new Book();
    }
}
```

###### 演示

``` java
@Controller
@RequestMapping(value = "/demo")
public class Demo {

    @Autowired
    private RedLamp redLamp;

    @RequestMapping(value = "/demo", method = RequestMethod.GET)
    @ResponseBody
    public void demo() {
        redLamp.shine();
    }
}
```

###### 输出

``` text
开始记录
发出红光
记录完成
```

###### 切面优化

切面可以将切点表达式提取出来

``` java
@Aspect
public class Book {

    @Pointcut("execution(* demo.Lamp.shine(..))")
    public void pointCut() {

    }

    @Before("pointCut()")
    public void startRecord() {
        System.out.println("开始记录");
    }

    @After("pointCut()")
    public void endRecord() {
        System.out.println("记录完成");
    }
}
```

也可以使用环绕通知

``` java
@Aspect
public class Book {

    @Pointcut("execution(* demo.Lamp.shine(..))")
    public void pointCut() {

    }

    @Around("pointCut()")
    public void aroundRecord(ProceedingJoinPoint proceedingJoinPoint) {

        try {
            System.out.println("开始记录");
            proceedingJoinPoint.proceed();
            System.out.println("记录完成");
        } catch (Throwable throwable) {
            System.out.println("记录异常");
        }
    }
}
```

##### 注解实现场景二

对灯每次发光的时间进行记录。

###### Lamp接口及其实现类

``` java
public interface Lamp {

    void shine(int minute);
}
```

``` java
public class RedLamp implements Lamp {
	
    @Override
    public void shine(int minute) {
        System.out.println("发出红光" + String.valueOf(minute) + "分钟");
    }
}
```

###### 定义切面

``` java
@Aspect
public class Book {

    @Pointcut("execution(* demo.Lamp.shine(..)) && args (minute)")
    public void pointCut(int minute) {

    }

    @Around("pointCut(minute)")
    public void aroundRecord(ProceedingJoinPoint proceedingJoinPoint, int minute) {

        try {
            System.out.println("开始记录，持续" + minute + "分钟");
            proceedingJoinPoint.proceed();
            System.out.println("记录完成");
        } catch (Throwable throwable) {
            System.out.println("记录异常");
        }
    }
}
```

###### 开启切面

``` java
@Configuration
@EnableAspectJAutoProxy
public class Config {

    @Bean
    public RedLamp redLamp() {
        return new RedLamp();
    }

    @Bean
    public Book book() {
        return new Book();
    }
}
```

###### 演示

``` java
@Controller
@RequestMapping(value = "/demo")
public class Demo {

    @Autowired
    private RedLamp redLamp;

    @RequestMapping(value = "/demo", method = RequestMethod.GET)
    @ResponseBody
    public void demo() {
        redLamp.shine(5);
    }
}
```

###### 输出

``` text
开始记录，持续5分钟
发出红光5分钟
记录完成
```

##### 注解实现场景三

使灯自己记录自己的时间

###### Lamp接口及其实现类

``` java
public interface Lamp {

    void shine(int minute);
}
```

``` java
public class RedLamp implements Lamp {

    @Override
    public void shine(int minute) {
        System.out.println("发出红光" + String.valueOf(minute) + "分钟");
    }
}
```

###### Book接口及其实现类

``` java
public interface Book {

    void startRecord(int minute);

    void endRecord();
}
```

``` java
public class LampBook implements Book {

    public void startRecord(int minute) {
        System.out.println("开始记录，持续" + minute + "分钟");
    }

    public void endRecord() {
        System.out.println("记录完成");
    }
}
```

###### 定义切面

``` java
@Aspect
public class BookIntroducer {

    @DeclareParents(value = "demo.RedLamp", defaultImpl = LampBook.class)
    public static Book book;
}
```

value也可以用+号表示统配所有该类型的，例如value = "demo.Lamp+"，加号表示所有Lamp类型的。

###### 开启切面

``` java
@Configuration
@EnableAspectJAutoProxy
@ComponentScan
@EnableAutoConfiguration(exclude = AopAutoConfiguration.class)
public class Config {

    @Bean
    public RedLamp redLamp() {
        return new RedLamp();
    }

    @Bean
    public LampBook lampBook() {
        return new LampBook();
    }

    @Bean
    public BookIntroducer bookIntroducer() {
        return new BookIntroducer();
    }
}
```

###### 演示

``` java
@Controller
@RequestMapping(value = "/demo")
public class Demo implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }

    @RequestMapping(value = "/demo", method = RequestMethod.GET)
    @ResponseBody
    public void demo() {

        Lamp lamp = applicationContext.getBean("redLamp", Lamp.class);
        Book book = applicationContext.getBean("redLamp", Book.class);
        book.startRecord(5);
        lamp.shine(5);
        book.endRecord();
    }
}
```

###### 输出

``` text
开始记录，持续5分钟
发出红光5分钟
记录完成
```

##### XML实现场景一

对灯每次发光进行记录。

###### Lamp接口及其实现类

``` java
public interface Lamp {

    void shine();
}
```

``` java
public class RedLamp implements Lamp {

    @Override
    public void shine() {
        System.out.println("发出红光");
    }
}
```

###### 切面Bean

``` java
public class Book {

    public void startRecord() {
        System.out.println("开始记录");
    }

    public void endRecord() {
        System.out.println("记录完成");
    }
}
```

###### xml配置

``` xml
<bean id="redLamp" class="demo.RedLamp"/>
<bean id="book" class="demo.Book"/>

<aop:aspectj-autoproxy/>
<aop:config>
    <aop:aspect ref="book">
        <aop:before pointcut="execution(* demo.Lamp.shine(..))" method="startRecord"/>
        <aop:after pointcut="execution(* demo.Lamp.shine(..))" method="endRecord"/>
    </aop:aspect>
</aop:config>
```

###### 演示

启动类需引入XML配置

``` java
@ImportResource("classpath:application-context.xml")
```

``` java
@Controller
@RequestMapping(value = "/demo")
public class Demo {

    @Autowired
    private RedLamp redLamp;

    @RequestMapping(value = "/demo", method = RequestMethod.GET)
    @ResponseBody
    public void demo() {
        redLamp.shine();
    }
}
```

###### 输出

``` text
开始记录
发出红光
记录完成
```

###### xml配置优化

配置可以将切点表达式提取出来

``` xml
<bean id="redLamp" class="demo.RedLamp"/>
<bean id="book" class="demo.Book"/>

<aop:aspectj-autoproxy/>
<aop:config>
    <aop:aspect ref="book">

        <aop:pointcut id="bookPointcut" expression="execution(* demo.Lamp.shine(..))"/>

        <aop:before pointcut-ref="bookPointcut" method="startRecord"/>
        <aop:after pointcut-ref="bookPointcut" method="endRecord"/>
    </aop:aspect>
</aop:config>
```

也可以使用环绕通知

``` java
public class Book {

    public void aroundRecord(ProceedingJoinPoint proceedingJoinPoint) {

        try {
            System.out.println("开始记录");
            proceedingJoinPoint.proceed();
            System.out.println("记录完成");
        } catch (Throwable throwable) {
            System.out.println("记录异常");
        }
    }
}
```

``` xml
<bean id="redLamp" class="demo.RedLamp"/>
<bean id="book" class="demo.Book"/>

<aop:aspectj-autoproxy/>
<aop:config>
    <aop:aspect ref="book">

        <aop:pointcut id="bookPointcut" expression="execution(* demo.Lamp.shine(..))"/>

        <aop:around pointcut-ref="bookPointcut" method="aroundRecord"/>
    </aop:aspect>
</aop:config>
```

##### XML实现场景二

对灯每次发光的时间进行记录。

###### Lamp接口及其实现类

``` java
public interface Lamp {

    void shine(int minute);
}
```

``` java
public class RedLamp implements Lamp {

    @Override
    public void shine(int minute) {
        System.out.println("发出红光" + String.valueOf(minute) + "分钟");
    }
}
```

###### 切面Bean

``` java
public class Book {

    public void aroundRecord(ProceedingJoinPoint proceedingJoinPoint, int minute) {

        try {
            System.out.println("开始记录，持续" + minute + "分钟");
            proceedingJoinPoint.proceed();
            System.out.println("记录完成");
        } catch (Throwable throwable) {
            System.out.println("记录异常");
        }
    }
}
```

###### xml配置

``` xml
<bean id="redLamp" class="demo.RedLamp"/>
<bean id="book" class="demo.Book"/>

<aop:aspectj-autoproxy/>
<aop:config>
    <aop:aspect ref="book">

        <aop:pointcut id="bookPointcut" expression="execution(* demo.Lamp.shine(..)) and args (minute)"/>

        <aop:around pointcut-ref="bookPointcut" method="aroundRecord"/>
    </aop:aspect>
</aop:config>
```

###### 演示

启动类需引入XML配置

``` java
@ImportResource("classpath:application-context.xml")
```

``` java
@Controller
@RequestMapping(value = "/demo")
public class Demo {

    @Autowired
    private RedLamp redLamp;

    @RequestMapping(value = "/demo", method = RequestMethod.GET)
    @ResponseBody
    public void demo() {
        redLamp.shine(5);
    }
}
```

###### 输出

``` text
开始记录，持续5分钟
发出红光5分钟
记录完成
```

##### XML实现场景三

使灯自己记录自己的时间

###### Lamp接口及其实现类

``` java
public interface Lamp {

    void shine(int minute);
}
```

``` java
public class RedLamp implements Lamp {

    @Override
    public void shine(int minute) {
        System.out.println("发出红光" + String.valueOf(minute) + "分钟");
    }
}
```

###### Book接口及其实现类

``` java
public interface Book {

    void startRecord(int minute);

    void endRecord();
}
```

``` java
public class LampBook implements Book {

    public void startRecord(int minute) {
        System.out.println("开始记录，持续" + minute + "分钟");
    }

    public void endRecord() {
        System.out.println("记录完成");
    }
}
```

###### xml配置

``` xml
<bean id="redLamp" class="demo.RedLamp"/>
<bean id="lampBook" class="demo.LampBook"/>

<aop:aspectj-autoproxy/>
<aop:config>
    <aop:aspect>
        <aop:declare-parents types-matching="demo.RedLamp" implement-interface="demo.Book" default-impl="demo.LampBook"/>
    </aop:aspect>
</aop:config>
```

###### 演示

启动类需引入XML配置

``` java
@ImportResource("classpath:application-context.xml")
```

``` java
@Controller
@RequestMapping(value = "/demo")
public class Demo implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }

    @RequestMapping(value = "/demo", method = RequestMethod.GET)
    @ResponseBody
    public void demo() {

        Lamp lamp = applicationContext.getBean("redLamp", Lamp.class);
        Book book = applicationContext.getBean("redLamp", Book.class);
        book.startRecord(5);
        lamp.shine(5);
        book.endRecord();
    }
}
```

###### 输出

``` text
开始记录，持续5分钟
发出红光5分钟
记录完成
```