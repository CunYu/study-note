本文从源码的角度上分析了Spring Boot的启动流程。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### Spring Boot 启动类

* Spring boot启动类相关代码

``` java
@SpringBootApplication
public class SpringBootDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootDemoApplication.class, args);
    }
}
```
* Spring boot启动类分析

Spring boot启动主要分为俩部分：@SpringBootApplication和SpringApplication的run方法。

### @SpringBootApplication

* @SpringBootApplication 相关源码

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	// 省略

}
```

* @SpringBootApplication 分析

@SpringBootApplication包含了@SpringBootConfiguration, @EnableAutoConfiguration, @ComponentScan。

@SpringBootConfiguration包含了@Configuration。

@EnableAutoConfiguration根据配置的jar依赖自动配置spring application。

@ComponentScan自动获取所有的spring组件，包含@Configuration声明的类。

### SpringApplication的run方法

* run方法相关源码

``` java
public static ConfigurableApplicationContext run(Class<?> primarySource,
		String... args) {
	return run(new Class<?>[] { primarySource }, args);
}

public static ConfigurableApplicationContext run(Class<?>[] primarySources,
		String[] args) {
	return new SpringApplication(primarySources).run(args);
}
```

* run方法分析

SpringApplication的run方法本质上分为俩步。

1. 新建一个SpringApplication实例

新建SpringApplication实例相关源码

``` java
public SpringApplication(Class<?>... primarySources) {
	this(null, primarySources);
}

@SuppressWarnings({ "unchecked", "rawtypes" })
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	this.resourceLoader = resourceLoader;
	Assert.notNull(primarySources, "PrimarySources must not be null");
	this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
	// 1.1 判断webApplication类型
	this.webApplicationType = deduceWebApplicationType();
	// 1.2 设置构造器
	setInitializers((Collection) getSpringFactoriesInstances(
			ApplicationContextInitializer.class));
	// 1.3 设置监听器
	setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
	// 1.4 推断应用主类
	this.mainApplicationClass = deduceMainApplicationClass();
}
```

1.1 判断webApplication类型

判断webApplication类型相关源码

``` java
private WebApplicationType deduceWebApplicationType() {
	// ClassUtils.isPresent 判断用户提供的类是否存在及被加载
	if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
			&& !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)
			&& !ClassUtils.isPresent(JERSEY_WEB_ENVIRONMENT_CLASS, null)) {
		return WebApplicationType.REACTIVE;
	}
	for (String className : WEB_ENVIRONMENT_CLASSES) {
		if (!ClassUtils.isPresent(className, null)) {
			return WebApplicationType.NONE;
		}
	}
	return WebApplicationType.SERVLET;
}

private static final String REACTIVE_WEB_ENVIRONMENT_CLASS = "org.springframework."
			+ "web.reactive.DispatcherHandler";

private static final String MVC_WEB_ENVIRONMENT_CLASS = "org.springframework."
			+ "web.servlet.DispatcherServlet";

private static final String JERSEY_WEB_ENVIRONMENT_CLASS = "org.glassfish.jersey.server.ResourceConfig";

private static final String[] WEB_ENVIRONMENT_CLASSES = { "javax.servlet.Servlet",
			"org.springframework.web.context.ConfigurableWebApplicationContext" };

public enum WebApplicationType {

	/**
	 * The application should not run as a web application and should not start an
	 * embedded web server.
	 */
	NONE,

	/**
	 * The application should run as a servlet-based web application and should start an
	 * embedded servlet web server.
	 */
	SERVLET,

	/**
	 * The application should run as a reactive web application and should start an
	 * embedded reactive web server.
	 */
	REACTIVE

}
```

1.2 设置构造器

设置构造器相关源码

``` java
public void setInitializers(
		Collection<? extends ApplicationContextInitializer<?>> initializers) {
	this.initializers = new ArrayList<>();
	this.initializers.addAll(initializers);
}

private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
	return getSpringFactoriesInstances(type, new Class<?>[] {});
}

private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
		Class<?>[] parameterTypes, Object... args) {
	// 获取类加载器
	ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
	// Use names and ensure unique to protect against duplicates
	Set<String> names = new LinkedHashSet<>(
			// 通过参数中的类加载器，从META-INF/spring.factories这个路径下，获得参数中指定类型的工厂实现类全路径。
			SpringFactoriesLoader.loadFactoryNames(type, classLoader));
	// 根据names中的类的全路径，对应实例化类
	List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
			classLoader, args, names);
	// 排序
	AnnotationAwareOrderComparator.sort(instances);
	return instances;
}

private <T> List<T> createSpringFactoriesInstances(Class<T> type,
		Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args,
		Set<String> names) {
	List<T> instances = new ArrayList<>(names.size());
	for (String name : names) {
		try {
			Class<?> instanceClass = ClassUtils.forName(name, classLoader);
			// 判断instanceClass是否按类型匹配type
			Assert.isAssignable(type, instanceClass);
			// 根据parameterTypes中指定的参数类型及参数顺序获取与传入对象的构造函数相对应的构造器
			Constructor<?> constructor = instanceClass
					.getDeclaredConstructor(parameterTypes);
			// 根据给定的构造器去实例化类
			T instance = (T) BeanUtils.instantiateClass(constructor, args);
			instances.add(instance);
		}
		catch (Throwable ex) {
			throw new IllegalArgumentException(
					"Cannot instantiate " + type + " : " + name, ex);
		}
	}
	return instances;
}
```

1.3 设置监听器

设置监听器相关源码

``` java
public void setListeners(Collection<? extends ApplicationListener<?>> listeners) {
	this.listeners = new ArrayList<>();
	this.listeners.addAll(listeners);
}

// private <T> Collection<T> getSpringFactoriesInstances(Class<T> type)方法前文已给出
```

1.4 推断应用主类

推断应用主类相关源码

``` java
private Class<?> deduceMainApplicationClass() {
	try {
		StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
		for (StackTraceElement stackTraceElement : stackTrace) {
			if ("main".equals(stackTraceElement.getMethodName())) {
				return Class.forName(stackTraceElement.getClassName());
			}
		}
	}
	catch (ClassNotFoundException ex) {
		// Swallow and continue
	}
	return null;
}
```

2. 调用SpringApplication实例的run方法

* SpringApplication实例run方法源码

``` java
public ConfigurableApplicationContext run(String... args) {
	// 2.1 计时工具
	StopWatch stopWatch = new StopWatch();
	stopWatch.start();
	ConfigurableApplicationContext context = null;
	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
	// 2.2 设置无图形界面
	configureHeadlessProperty();
	// 2.3 获取spring运行监听器
	SpringApplicationRunListeners listeners = getRunListeners(args);
	listeners.starting();
	try {
		ApplicationArguments applicationArguments = new DefaultApplicationArguments(
				args);
		// 2.4 环境准备及配置
		ConfigurableEnvironment environment = prepareEnvironment(listeners,
				applicationArguments);
		configureIgnoreBeanInfo(environment);
		// 2.5 spring boot项目启动打印
		Banner printedBanner = printBanner(environment);
		// 2.6 获取spring context
		context = createApplicationContext();
		// 2.7 获取异常报告器
		exceptionReporters = getSpringFactoriesInstances(
				SpringBootExceptionReporter.class,
				new Class[] { ConfigurableApplicationContext.class }, context);
		// 2.8 spring context处理
		prepareContext(context, environment, listeners, applicationArguments,
				printedBanner);
		refreshContext(context);
		afterRefresh(context, applicationArguments);
		// 2.9 停止计时器
		stopWatch.stop();
		// 2.10 应用启动日志打印
		if (this.logStartupInfo) {
			new StartupInfoLogger(this.mainApplicationClass)
					.logStarted(getApplicationLog(), stopWatch);
		}
		listeners.started(context);
		// 2.11 调用Runners
		// Runners是ApplicationRunner CommandLineRunner的实现类
		callRunners(context, applicationArguments);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, listeners);
		throw new IllegalStateException(ex);
	}

	try {
		listeners.running(context);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, null);
		throw new IllegalStateException(ex);
	}
	return context;
}
```

2.2 设置无图形界面

设置无图形界面相关源码

``` java
private void configureHeadlessProperty() {
	System.setProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, System.getProperty(
			SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, Boolean.toString(this.headless)));
}

private static final String SYSTEM_PROPERTY_JAVA_AWT_HEADLESS = "java.awt.headless";
```

2.3 获取spring运行监听器

获取spring运行监听器相关源码

``` java
private SpringApplicationRunListeners getRunListeners(String[] args) {
	Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
	return new SpringApplicationRunListeners(logger, getSpringFactoriesInstances(
			SpringApplicationRunListener.class, types, this, args));
}

// private <T> Collection<T> getSpringFactoriesInstances(Class<T> type)方法前文已给出
```

2.4 环境准备及配置

环境准备及配置相关源码

``` java
private ConfigurableEnvironment prepareEnvironment(
		SpringApplicationRunListeners listeners,
		ApplicationArguments applicationArguments) {
	// Create and configure the environment
	ConfigurableEnvironment environment = getOrCreateEnvironment();
	configureEnvironment(environment, applicationArguments.getSourceArgs());
	listeners.environmentPrepared(environment);
	bindToSpringApplication(environment);
	if (this.webApplicationType == WebApplicationType.NONE) {
		environment = new EnvironmentConverter(getClassLoader())
				.convertToStandardEnvironmentIfNecessary(environment);
	}
	ConfigurationPropertySources.attach(environment);
	return environment;
}

private void configureIgnoreBeanInfo(ConfigurableEnvironment environment) {
	if (System.getProperty(
			CachedIntrospectionResults.IGNORE_BEANINFO_PROPERTY_NAME) == null) {
		Boolean ignore = environment.getProperty("spring.beaninfo.ignore",
				Boolean.class, Boolean.TRUE);
		System.setProperty(CachedIntrospectionResults.IGNORE_BEANINFO_PROPERTY_NAME,
				ignore.toString());
	}
}

public static final String IGNORE_BEANINFO_PROPERTY_NAME = "spring.beaninfo.ignore";
```

2.5 spring boot项目启动打印

spring boot项目启动打印相关源码

``` java
private Banner printBanner(ConfigurableEnvironment environment) {
	if (this.bannerMode == Banner.Mode.OFF) {
		return null;
	}
	ResourceLoader resourceLoader = (this.resourceLoader != null ? this.resourceLoader
			: new DefaultResourceLoader(getClassLoader()));
	SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(
			resourceLoader, this.banner);
	if (this.bannerMode == Mode.LOG) {
		return bannerPrinter.print(environment, this.mainApplicationClass, logger);
	}
	return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
}
```

2.6 获取spring context

获取spring context相关源码

``` java
protected ConfigurableApplicationContext createApplicationContext() {
	Class<?> contextClass = this.applicationContextClass;
	if (contextClass == null) {
		try {
			switch (this.webApplicationType) {
			case SERVLET:
				contextClass = Class.forName(DEFAULT_WEB_CONTEXT_CLASS);
				break;
			case REACTIVE:
				contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
				break;
			default:
				contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
			}
		}
		catch (ClassNotFoundException ex) {
			throw new IllegalStateException(
					"Unable create a default ApplicationContext, "
							+ "please specify an ApplicationContextClass",
					ex);
		}
	}
	return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
}

/**
 * The class name of application context that will be used by default for web
 * environments.
 */
public static final String DEFAULT_WEB_CONTEXT_CLASS = "org.springframework.boot."
		+ "web.servlet.context.AnnotationConfigServletWebServerApplicationContext";

/**
 * The class name of application context that will be used by default for reactive web
 * environments.
 */
public static final String DEFAULT_REACTIVE_WEB_CONTEXT_CLASS = "org.springframework."
		+ "boot.web.reactive.context.AnnotationConfigReactiveWebServerApplicationContext";

/**
 * The class name of application context that will be used by default for non-web
 * environments.
 */
public static final String DEFAULT_CONTEXT_CLASS = "org.springframework.context."
		+ "annotation.AnnotationConfigApplicationContext";
```

2.7 获取异常报告器

获取异常报告器相关源码前文已给出。

2.8 spring context处理

spring context处理相关源码

``` java
private void prepareContext(ConfigurableApplicationContext context,
		ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
		ApplicationArguments applicationArguments, Banner printedBanner) {
	// 设置环境
	context.setEnvironment(environment);
	// ApplicationContext后置处理
	postProcessApplicationContext(context);
	// 在context刷新之前应用构造器
	applyInitializers(context);
	// 调用监听器的contextPrepared方法
	listeners.contextPrepared(context);
	if (this.logStartupInfo) {
		logStartupInfo(context.getParent() == null);
		logStartupProfileInfo(context);
	}

	// 添加启动过程中的俩个特殊单例
	// Add boot specific singleton beans
	context.getBeanFactory().registerSingleton("springApplicationArguments",
			applicationArguments);
	if (printedBanner != null) {
		context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
	}
	// 获取所有的资源
	// Load the sources
	Set<Object> sources = getAllSources();
	Assert.notEmpty(sources, "Sources must not be empty");
	// Application context加载beans
	load(context, sources.toArray(new Object[0]));
	// 调用监听器的contextLoaded方法
	listeners.contextLoaded(context);
}

private void refreshContext(ConfigurableApplicationContext context) {
	// 刷新底层的application context
	refresh(context);
	if (this.registerShutdownHook) {
		try {
			context.registerShutdownHook();
		}
		catch (AccessControlException ex) {
			// Not allowed in some environments.
		}
	}
}

// Context刷新后调用
protected void afterRefresh(ConfigurableApplicationContext context,
		ApplicationArguments args) {
}
```

2.11 调用Runners

调用Runners相关源码

``` java
private void callRunners(ApplicationContext context, ApplicationArguments args) {
	List<Object> runners = new ArrayList<>();
	runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
	runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
	AnnotationAwareOrderComparator.sort(runners);
	for (Object runner : new LinkedHashSet<>(runners)) {
		if (runner instanceof ApplicationRunner) {
			callRunner((ApplicationRunner) runner, args);
		}
		if (runner instanceof CommandLineRunner) {
			callRunner((CommandLineRunner) runner, args);
		}
	}
}
```
