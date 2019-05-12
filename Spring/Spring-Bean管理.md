本文介绍了Spring Bean的相关知识。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### Spring Bean

Spring的目标是让Java开发变得更加简单，其用Spring容器来负责Bean的创建，装配并管理Bean的整个生命周期。其通过DI（依赖注入）来管理Bean之间的关系，极大的降低了系统的耦合度，与此同时，通过Spring容器主动去管理Bean，会使我们节省下很大的开销，（例如，我们需要一个单例Bean的时候，需要做很多工作，而Spring中，其管理的Bean默认为单例的，我们从容器中获取的Bean就是单例的，你只需要使用即可。）

### Spring IOC容器

Spring容器负责Bean的创建，装配并管理Bean的整个生命周期。

因为Spring容器是用DI来管理Bean之间的关系，所以Spring容器是IOC容器。

在Spring中，Spring容器不是唯一的，BeanFactory是Spring中最简单的容器，提供了基本的DI支持，我们平时最常使用的是ApplicationContext，其基于BeanFactory构建，并提供应用框架级别的服务支持。

ApplicationContext常见构建方法

|名称|功能|
|:----|:----|
|AnnotationConfigApplicationContext|从Java配置类中加载Spring ApplicationContext|
|AnnotationConfigWebApplicationContext|从Java配置类中加载Spring Web ApplicationContext|
|ClassPathXmlApplicationContext|从类路径下的xml文件加载ApplicationContext|
|FileSystemXmlApplicationContext|从文件系统下的xml文件加载ApplicationContext|
|XmlWebApplicationContext|从Web应用下的xml文件加载ApplicationContext|

### Bean 生命周期

1 Spring实例化Bean。

2 Spring将值及Bean的引用注入到Bean中。

3 判断Bean是否实现了BeanNameAware接口，如实现，则调用setBeanName方法将bean的ID传入。

4 判断Bean是否实现了BeanFactoryAware接口，如实现，则调用setBeanFactory方法将BeanFactory容器实例传入。

5 判断Bean是否实现了ApplicationContextAware接口，如实现，则调用setApplicationContext方法将ApplicationContext容器实例传入。

6 判断Bean是否实现了BeanPostProcessor接口，如实现，则调用postProcessBeforeInitialization方法。

7 判断Bean是否实现了InitializingBean接口，如实现，则调用afterPropertiesSet方法，即使Bean使用init-method声明了初始化方法，该方法也会被调用。

8 判断Bean是否实现了BeanPostProcessor接口，如实现，则调用postProcessAfterInitialization方法。

9 Bean已创建完毕并存放在ApplicationContext中。

10 容器关闭，销毁Bean时，判断Bean是否实现了DisPosableBean接口，如实现，则调用destroy方法，即使Bean使用destroy-method声明了销毁方法，该方法也会被调用。

### Bean 装配

##### 自动装配

* @Component

该注解声明的类会作为组件类，Spring会为这个类创建Bean，并将该类注册到Spring容器中，Bean ID为首字母小写的类名，或者在注解中声明。

声明了组件，需要开启组件扫描功能才行。

开启组件扫描有两组方式，注解和Xml配置。

``` java
@ComponentScan
```

``` xml
<context:component-scan base-package="com.xxx.Xxx"/>
```

组件扫描需要设置扫描的基础包，也可以设置扫描的基础类或接口，当使用注解或设置扫描的基础类或接口时，该类或接口所在的包会为扫描的基础包。

* @Named

该注解是Java依赖注入规范中提供的。其作用大体上和@Component一致，大多数场景下，是可以相互替换的。

* @Autowired

该注解可以用到类的属性和方法上，当用在属性上时，Spring会自动去为该属性从Spring容器中寻找其所需要的依赖并满足其依赖的需求。当用在方法上时，Spring会自动去为该方法参数从Spring容器中寻找其所需要的依赖并满足其依赖的需求。

@Autowired优先根据类型寻找所需的依赖，当按类型寻找无法满足依赖的需求时，再根据beanId寻找所需的依赖。

当Spring容器中没有其所需的依赖或者发现多个可以满足其需求的依赖，Spring会抛出异常，@Autowired(required = false)可以告诉Spring该依赖不是必须的，但是这样会带来一定的风险。

* @Inject

该注解是Java依赖注入规范中提供的。其作用大体上和@Autowired一致，大多数场景下，是可以相互替换的。

##### Java 装配

* @Configuration

该注解声明的类会作为配置类。

* @Bean

该注解用在配置类中的方法上，该注解告诉Spring该方法返回的对象要注册到Spring容器中，Bean ID为方法名，或者在注解声明。

该注解声明方法有参数时，Spring会自动去为该方法参数从Spring容器中寻找其所需要的依赖并满足其依赖的需求。

@Bean优先根据类型寻找所需的依赖，当按类型寻找无法满足依赖的需求时，再根据beanId寻找所需的依赖。

* @Import(Xxx.class)

该注解可引入配置类。

* @ImportResource("classpath:xxx.xml")

该注解可引入xml配置文件。

##### Xml 装配

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="xxxOne" class="com.xxx.XxxOne">
        <constructor-arg ref="xxxTwo"/>
        <constructor-arg value="demo"/>
        <constructor-arg><null/></constructor-arg>
        <constructor-arg>
            <list>
                <value>demoOne</value>
                <value>demoTwo</value>
            </list>
        </constructor-arg>
    </bean>

    <bean id="xxxThree" class="com.xxx.XxxTwo">
        <property name="propertyOne" value="demo"/>
        <property name="propertyTwo" ref="xxxFour"/>
    </bean>
    
    <!--导入其他配置文件-->
    <import resource="xxx.xml"/>
</beans>
```

Spring同时也提供了c-命令空间，p-命名空间，util-命令空间等来实现Xml装配Bean。

### Bean 条件化声明

* @Conditional

该注解所声明的Bean只有在@Conditional注解给定的条件结果为true时创建。

@Conditional注解给定的条件必须为实现Condition接口的类，其matches方法返回的值作为其条件结果。

### Bean 装配冲突处理

Spring会根据Bean需求自动去满足Bean所需要的依赖，当发现多个依赖均能满足Bean的需求时，Spring就不知道该选择哪个依赖来满足Bean的需求，这时Spring会抛出异常。

* @Primary

该注解所声明的Bean会作为首选项，当发现多个依赖均能满足Bean的需求时，优先选择首选项。当发现多个依赖均能满足Bean的需求且都标注@Primary时，Spring不知道该选择哪个依赖来满足Bean的需求，这时Spring会抛出异常。

该功能也可以使用primary="true"Xml配置实现。

* @Qualifier

该注解用来表示限定符。

当声明在Bean创建时，会被Bean声明一个限定符，默认限定符与Bean ID相同，也可以注解中声明。

当声明在Bean装配时，表示只有满足该限定符的Bean才能被装配。

### Bean 作用域

|作用域|常量|说明|
|:----|:----|:----|
|单例（Singleton）|ConfigurableBeanFactory.SCOPE_SINGLETON|Spring只创建Bean的一个实例，Spring中所有使用该Bean的地方都是使用的同一个实例|
|原型（Prototype）|ConfigurableBeanFactory.SCOPE_PROTOTYPE|每次注入或者通过ApplicationContext获取Bean时，都会创建一个新的Bean实例|
|会话（Session）|WebApplicationContext.SCOPE_SESSION|Web应用中，每个会话都会创建一个Bean实例|
|请求（Request）|WebApplicationContext.SCOPE_REQUEST|Web应用中，每个请求都会创建一个Bean实例|

Bean作用域可用@Scope注解和scopeXml配置实现。

会话或者请求作用域Bean需要使用代理模式，假设一个单例Bean使用到了会话或者请求作用域Bean，当单例Bean创建时，会话或请求作用域的Bean不存在，因此需要使用代理模式。

@Scope注解的proxyMode = ScopedProxyMode.INTERFACES属性可用来指出对应的代理模式。

注解
``` java
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.INTERFACES)

@Bean
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.INTERFACES)
```

Xml配置

``` xml
<bean id="xxx" class="com.xxx.Xxx" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>
```