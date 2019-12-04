### 概述

循环依赖是指依赖形成了一个环，例如A依赖B，B依赖C，C依赖A，这样会在初始化时如果不处理的话会发生OOM。

### Spring循环依赖分类

##### 构造器循环依赖

构造器循环依赖是指构造器形成循环依赖，例如A构造器依赖B，B构造器依赖C，C构造器依赖A。

##### 属性成员循环依赖

属性成员循环依赖是指对象中的属性成员形成循环依赖，例如A中的成员变量为B，B中的成员变量为C，C中的成员变量为A。

### Spring循环依赖处理方案

Spring可以处理属性成员循环依赖，但是只能处理单例作用域的属性成员循环依赖，因为Spring只对单例作用域的Bean做了缓存。

##### 构造器循环依赖

构造器循环依赖Spring是无法处理的，Spring会将创建中的对象存到当前创建Bean池中，创建完毕后移除。

A创建时，会将自己存到当前创建Bean池中，因为其构造器依赖B，这时会去创建B，B创建时，会将自己存到当前创建Bean池中，因为其构造器依赖C，这时会去创建C，C创建时，会将自己存到当前创建Bean池中，因为其构造其依赖A，这时会去创建A，A创建时需要将自己存入当前创建Bean池中，但是A现在已经在当前创建Bean池中存在了，这时其会抛出异常。

##### 属性成员循环依赖

Spring采用提前暴露对象（调用构造器完成但依赖关系还没好）的方法来解决单例作用域属性成员循环依赖，Spring Bean创建时，其会先通过反射创建对象，然后再满足其依赖，对象一旦创建（调用构造器完成）完，其就会暴露出来。

A对象创建完后，发现其需要B依赖，其会去创建B对象，创建完B后，发现其需要C依赖，其会去创建C对象，创建完C后，发现其需要A依赖，这时其可以找到A对象，满足需要，C创建完毕，然后B对象所需依赖满足，B创建完毕，这时A对象所需依赖满足，A创建完毕。

### 三级缓存

DefaultSingletonBeanRegistry部分源码

``` java
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
```

``` java
/** Names of beans that are currently in creation. */
private final Set<String> singletonsCurrentlyInCreation =
        Collections.newSetFromMap(new ConcurrentHashMap<>(16));
```

``` java
@Override
@Nullable
public Object getSingleton(String beanName) {
	return getSingleton(beanName, true);
}

/**
 * Return the (raw) singleton object registered under the given name.
 * <p>Checks already instantiated singletons and also allows for an early
 * reference to a currently created singleton (resolving a circular reference).
 * @param beanName the name of the bean to look for
 * @param allowEarlyReference whether early references should be created or not
 * @return the registered singleton object, or {@code null} if none found
 */
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
	Object singletonObject = this.singletonObjects.get(beanName);
	if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
		synchronized (this.singletonObjects) {
			singletonObject = this.earlySingletonObjects.get(beanName);
			if (singletonObject == null && allowEarlyReference) {
				ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
				if (singletonFactory != null) {
					singletonObject = singletonFactory.getObject();
					this.earlySingletonObjects.put(beanName, singletonObject);
					this.singletonFactories.remove(beanName);
				}
			}
		}
	}
	return singletonObject;
}
```

当对象开始创建时，便会存放在singletonsCurrentlyInCreation中，直至创建完成，才会移除。

获取对象时，先从singletonObjects中获取，获取不到时再从earlySingletonObjects中获取，再获取不到时再从singletonFactories中获取。

Spring Bean在创建完对象（调用完构造器）后，便会将其存放在singletonFactories中。