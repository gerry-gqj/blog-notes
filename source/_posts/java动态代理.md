---
title: java动态代理
typora-root-url: ../
date: 2022-11-23 13:50:00
tags:
 - java
 - 设计模式
categories:
 - java
---



JDK提供了`java.lang.reflect.InvocationHandler`接口和 `java.lang.reflect.Proxy`类，这两个类相互配合，入口是Proxy，所以我们先聊它。



## `Proxy`代理类



准备工作

```java
public interface IPeople {
    String getName(String name);
}

class IPeopleImpl implements IPeople{
    @Override
    public String getName(String name) {
        System.out.println("my name is: " + name);
        return name;
    }
}
```

```java
package com.qibria.demo_02;

import java.lang.reflect.*;

public class PeopleProxy<T> {

    public T target;

    public PeopleProxy(T target) {
        this.target = target;
    }

    public T getProxy_01() throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Class<?> aClass = Proxy.getProxyClass(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces());
        Constructor<?> constructor = aClass.getConstructor(InvocationHandler.class);
        return (T)constructor.newInstance(new Handler());
    }

    public T getProxy_02(){
        return (T) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                //target.getClass().getInterfaces(),
                new Class[] { IPeople.class },
                new Handler());
    }

    class Handler implements InvocationHandler{
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("\n===> before method");
            System.out.println("proxy: "+proxy.getClass());
            System.out.println("target: "+target.getClass());
            Object invoke = method.invoke(target, args);
            System.out.println("<=== after method");
            //return proxy;
            return invoke;
        }
    }

    public static void main(String[] args) throws InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
        IPeople iPeople = new IPeopleImpl();
        PeopleProxy<IPeople> peopleProxy = new PeopleProxy<>(iPeople);
        peopleProxy.getProxy_01().getName("aaa");
        peopleProxy.getProxy_02().getName("bbb");
        System.out.println("返回对象: "+peopleProxy.getProxy_01().getClass()); //返回对象: class com.sun.proxy.$Proxy0
    }
}


```



Proxy有个静态方法：`getProxyClass(ClassLoader, interfaces)`，只要你给它传入类加载器和一组接口，它就给你返回代理Class对象。

### `getProxyClass()`

```java
public T getProxy_01() throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
    Class<?> aClass = Proxy.getProxyClass(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces());
    Constructor<?> constructor = aClass.getConstructor(InvocationHandler.class);
    return (T)constructor.newInstance(new Handler());
}
```



实际编程中，一般不用`getProxyClass()`，而是使用Proxy类的另一个静态方法：`Proxy.newProxyInstance()`，直接返回代理实例，连中间得到代理Class对象的过程都帮你隐藏：

### `newProxyInstance()`

```java
public T getProxy_02(){
    return (T) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            new Handler());
}
```

>  `getProxy_01()`和`getProxy_02()`是完全等效的，一下就直接使用`getProxy_02()`作为演示



### `方法参数讲解`

`getProxyClass(ClassLoader, interfaces)`

`newProxyInstance(ClassLoader, interfaces, hander)`



- `ClassLoader`

加载器，直接获取代理对象的加载器作为参数即可



- `interfaces-Class[]`

上面是基于接口实现类从而实现动态代理功能，所以直接获取目标对象实现的接口的Class数组即可

```java
target.getClass().getInterfaces();
```

` java.lang---public Class<?>[] getInterfaces()` 方法返回的就是一个Class数组，符合预期



也可以`new Class[]{ type }`作为参数，数组里面的的`Class`必须是接口类型`(interface)`的，type必须是`interface`类型的`Class`，这样才能产生代理对象，否则会直接抛异常

修改第二个参数，效果也是完全等效的

```java
public T getProxy_02(){
    return (T) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            //target.getClass().getInterfaces(),
            new Class[] { IPeople.class },
            new Handler());
}
```





## `InvocationHandler`接口



根据代理Class的构造器创建对象时，需要传入`InvocationHandler`。每次调用代理对象的方法，最终都会调用`InvocationHandler`的`invoke()`方法：



代理对象的内部确实有个成员变量`invocationHandler`，而且代理对象的每个方法内部都会调用`handler.invoke()`！`InvocationHandler`对象成了代理对象和目标对象的桥梁，不像静态代理这么直接。



```java
class Handler implements InvocationHandler{
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return method.invoke(target,args);
    }
}
```

proxy是真实对象的真实代理对象，invoke方法可以返回调用代理对象方法的返回结果，也可以返回对象的真实代理对象（`com.sun.proxy.$Proxy0）`。

proxy参数是invoke方法的第一个参数，通常情况下我们都是返回真实对象方法的返回结果，但是我们也可以将proxy返回，proxy是真实对象的真实代理对象，我们可以通过这个返回对象对真实的对象做各种各样的操作。上面的例子就是返回对象，如果不是放回this对象会抛异常。

```java
public interface IPeople {
    IPeople getName(String name);
}

class IPeopleImpl implements IPeople{
    @Override
    public IPeople getName(String name) {
        System.out.println("my name is: " + name);
        System.out.println("this: "+this);
        System.out.println("super: "+ super.getClass().getName());
        return this;
    }
}
```



```java
public static void main(String[] args) throws InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
    IPeople iPeople = new IPeopleImpl();
    PeopleProxy peopleProxy = new PeopleProxy(iPeople);
    peopleProxy.getProxy_01().getName("aaa");
    peopleProxy.getProxy_02().getName("bbb");
    System.out.println("返回对象: "+peopleProxy.getProxy_01().getClass()); //返回对象: class com.sun.proxy.$Proxy0
}
```





## 一些简单的动态代理框架模型

```java
public interface IUser {
    String getName(String name);
}
```



```java
public interface IUserHandler {
    String setName(String name);
}
```

```java
public class IUserHandlerImpl implements IUserHandler {
    @Override
    public String setName(String name) {
        return name;
    }
}
```

```java
public interface ProxyCreator {

    Object createProxy(Class<?> type);

}
```

```java
public class ProxyCreatorImpl implements ProxyCreator {

    @Override
    public Object createProxy(Class<?> type) {
        
        IUserHandler handler = new IUserHandlerImpl();
        
        return Proxy.newProxyInstance(
                this.getClass().getClassLoader(),
                //直接根据接口创建出一个带有构造函数的代理对象, 该对象拥有接口的所有方法信息, 拿到方法信息后，然后自定义handler对象执行相应的逻辑
                new Class[]{ type },
                //必须使用接口实现类, 作用更多的是对现有的接口实现类进行修饰
                //type.getInterfaces(),
                new ProxyInvocationHandler(handler));
    }
}
```

```java
public class ProxyInvocationHandler implements InvocationHandler {

    IUserHandler handler;

    public ProxyInvocationHandler(IUserHandler handler) {
        this.handler = handler;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("==> method execute start");
        Object o;
        if (args[0] == "1"){
            o = handler.setName("张三");
        }else if (args[0]=="2"){
            o = handler.setName("里斯");
        }else {
            throw new NullPointerException("没有找到此用户");
        }
        System.out.println("<== method execute end");
        return o;
    }
}
```

```java
    public static void main(String[] args) {
        ProxyCreatorImpl proxyCreator = new ProxyCreatorImpl();
        IUser proxy = (IUser)proxyCreator.createProxy(IUser.class);
        String hhh = proxy.getName("2");
        System.out.println(hhh);
    }
```



## 基于`Spring FactoryBean`的动态代理框架封装









