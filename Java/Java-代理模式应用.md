本文介绍了代理模式在java中的应用并附有java代码实现demo。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 代理模式

代理模式为对象提供能控制对象本身的代理，操作代理就相当于操作对象。

### 示例场景

假设小赵是一家公司的老板，小钱是其手下的员工，有一个会议，小赵没时间参与，于是其委托小钱全权代表自己出席会议，小钱的决定就是自己的决定，这个场景中小钱其实就是老板的代理，其做的决定相对于老板做的决定。

### 静态代理

静态代理需要目标对象和代理对象实现相同的接口或类。

##### 示例

People接口

```
public interface People {

    void agree();

    void oppose();
}
```

Boss

```
public class Boss implements People {

    @Override
    public void agree() {
        System.out.println("同意");
    }

    @Override
    public void oppose() {
        System.out.println("反对");
    }
}
```

Boss代理

```
public class ProxyBoss implements People{

    private Boss boss;

    public ProxyBoss(Boss boss) {
        this.boss = boss;
    }

    @Override
    public void agree() {
        boss.agree();
    }

    @Override
    public void oppose() {
        boss.oppose();
    }
}
```

操作示例

```
Boss boss = new Boss();
People employee = new ProxyBoss(boss);
employee.agree();
employee.oppose();
```

输出

```
同意
反对
```

### 动态代理

动态代理需要用到JDK,其通过在内存中构建接口的实现类作为代理类来实现代理，动态代理又称为JDK代理或接口代理。

动态代理目标类必须实现接口。

##### 示例

People接口

```
public interface People {

    void agree();

    void oppose();
}
```

Boss

```
public class Boss implements People {

    @Override
    public void agree() {
        System.out.println("同意");
    }

    @Override
    public void oppose() {
        System.out.println("反对");
    }
}
```

Boss代理工厂

```
public class ProxyBossFactory {

    private Boss boss;

    public ProxyBossFactory(Boss boss) {
        this.boss = boss;
    }

    public Object getProxyInstance() {
        return Proxy.newProxyInstance(boss.getClass().getClassLoader(), boss.getClass().getInterfaces(), (proxy, method, args) -> {
            System.out.println("反射执行Boss方法开始");
            Object object = method.invoke(boss, args);
            System.out.println("反射执行Boss方法结束");
            return object;
        });
    }
}
```

操作示例

```
Boss boss = new Boss();
People employee = (People) new ProxyBossFactory(boss).getProxyInstance();
employee.agree();
employee.oppose();
```

输出

```
反射执行Boss方法开始
同意
反射执行Boss方法结束
反射执行Boss方法开始
反对
反射执行Boss方法结束
```

### cglib

cglib代理又称子类代理，其不需要目标类实现接口，也不会要求代理类实现接口，其通过在内存中构建目标类的子类作为代理类来实现代理。

引入依赖

```
compile 'cglib:cglib:3.2.9'
```

Boss

```
public class Boss {

    public void agree() {
        System.out.println("同意");
    }

    public void oppose() {
        System.out.println("反对");
    }
}
```

Boss代理工厂

```
public class ProxyBossFactory implements MethodInterceptor {

    private Boss boss;

    public ProxyBossFactory(Boss boss) {
        this.boss = boss;
    }

    public Object getProxyInstance() {

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(boss.getClass());
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("反射执行Boss方法开始");
        Object object = method.invoke(boss, args);
        System.out.println("反射执行Boss方法结束");
        return object;
    }
}
```

操作示例

```
Boss boss = new Boss();
Boss proxyBoss = (Boss) new ProxyBossFactory(boss).getProxyInstance();
proxyBoss.agree();
proxyBoss.oppose();
```

输出

```
反射执行Boss方法开始
同意
反射执行Boss方法结束
反射执行Boss方法开始
反对
反射执行Boss方法结束
```