---
title: 【Java基础系列】代理
categories: Java基础系列
tags: Java
abbrlink: 53c8b77c
date: 2024-02-26 12:48:41
---
# 什么是代理
   在编译时期无法确定需要实现哪个接口时，我们可以使用代理，在运行时创建实现一组给定接口的新类。对于编写应用程序的程序员来说这种场景比较少见，但对某些系统应用程序，代理带来的灵活性十分重要。

   总体来说，代理就是为某些对象的某种行为提供一个代理对象，并由代理对象完全控制该行为的实际执行。代理分为静态代理和动态代理两类，两者的主要区别就是代理类生成的时机，其中：

- 静态代理：在程序运行前，创建代理类，实现代理逻辑，编译时就已经实现了，编译完成后代理类成为一个实际的class文件。</br>
- 动态代理：在程序运行时，运用反射机制动态创建代理类，编译时没有实际的class文件，而是在运行时动态生成字节码。</br>

特别地，动态代理又有两种主要的实现方式，分别为：JDK 动态代理和 CGLIB 动态代理。
# 静态代理
   当我们需要在不修改目标对象的前提下，扩展目标对象的功能，就可以使用静态代理。例如：

我们有一个接口类：
``` java
public interface IHello {
    void sayHello();
}
```
有一个目标对象：
``` java
public class PersonDao implement IHello {

    @Override
    public void sayHello() {
        System.out.println("Hello world!");
    }
    
}
```
那么我们的静态代理对象可以这么构建，可以看出，静态Proxy相当于将要代理的对象封装了一层，重写并嵌套使用其所有的接口方法。
``` java
public class PersonProxy implement IHello {

    private IHello target；
    
    public PersonProxy(IHello target) {
        this.target = target;    
    }
    
    @Override
    public void sayHello() {
        ……
        target.sayHello();
        ……
    }
    
}
```
# 动态代理
   假如我们要构造一个类的对象，这个类实现了1个或多个接口，但在编译时期，可能并不知道这些接口是什么，这个时候就可以使用动态代理来解决我们的问题。动态代理有两大类：JDK和CGLIB，两者最大的不同是JDK需要代理的对象是基于接口实现的，而CGLIB作为第三方代码生成的类库，没有这种需求。

   动态代理基本会代理所有的代理对象的方法,对于从Object中继承的方法，JDK Proxy会把hashCode()、equals()、toString()这三个非接口方法转发给InvocationHandler，其余的Object方法则不会转发，例如getClass()、clone()方法。 

## JDK动态代理
   JDK 动态代理就是基于 JDK 实现的代理模式，主要运用了其拦截器和反射机制，其代理对象是由 JDK 动态生成的，而不像静态代理方式写死代理对象和被代理类。JDK 代理是不需要第三方库支持的，只需要 JDK 环境就可以进行代理，使用条件：
- 必须实现InvocationHandler
- 使用Proxy.newProxyInstance产生代理对象
- 被代理的对象必须实现一个或多个接口


> 使用 JDK 动态代理的五大步骤：</br>
> 1.通过实现InvocationHandler接口来定义自己的InvocationHandler</br>
> 2.通过Proxy.getProxyClass获得动态代理类</br>
> 3.通过反射机制获得代理类的构造方法，方法签名为getConstructor(InvocationHandler.class)</br>
> 4.通过构造函数获得代理对象并将自定义的InvocationHandler实例对象为参数传入</br>
> 5.通过代理对象调用目标方法。</br>

  接下来，我们就按上面的 5 个步骤，写一个 JDK 动态代理的示例。
- PersonProxyFactory，HelloImpl的代理类，这个代理类中需要实现InvocationHandler接口的invoke方法。
``` java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class PersonProxyFactory<T extends IHello> {

    // 要代理的对象
    private final T instance;

    public PersonProxyFactory(T instance) {
        this.instance = instance;
    }

    @SuppressWarnings("unchecked")
    public IHello createProxy() {
        InvocationHandler invocationHandler = invocationHandler();
        // 传入代理目标使用的类加载器、代理目标实现的接口类型、对应的事件处理器。
        return (IHello) Proxy.newProxyInstance(instance.getClass().getClassLoader(), instance.getClass().getInterfaces(), invocationHandler);
    }

    private InvocationHandler invocationHandler() {
        // 实现InvocationHandler接口的invoke方法
        return (proxy, method, args) -> {
            try {
                return method.invoke(instance, args);
            } catch (Exception e) {
                System.out.println(e.getMessage());
            }
            return null;
        };
    }
}
```
- MyProxyTest，测试类
``` java
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Proxy;

public class MyProxyTest {

    public static void main(String[] args) {
        PersonProxyFactory personProxyFactory = new PersonProxyFactory(new Person());
        personProxyFactory.createProxy().sayHello();
    }
    
}
```
## CGLIB动态代理
cglib (Code Generation Library )是一个第三方代码生成类库，运行时在内存中动态生成一个子类对象从而实现对目标对象功能的扩展。它广泛地被许多AOP的框架使用，例如Spring AOP和dynaop，为他们提供方法的interception（拦截）。

还是之前的几个例子，我们实现CGLIB的动态代理：
``` java
public class CGLibPersonProxyFactory<T extends IHello> implements MethodInterceptor {

    private final T target;

    public CGLibPersonProxyFactory(T instance) {
        this.target = instance;
    }

    public T createProxy() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(target.getClass());
        enhancer.setCallback(this);
        return (T) enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("CGLIB HELLO WORLD !");
        return methodProxy.invoke(target, objects);
    }
}
```

## 注意事项
- 在动态代理的过程中，如果被代理的对象的执行方法抛错，调用方会得到InvocationTargetException类的异常，这个异常封装了实际的问题，需要使用getTargetException()方法将内容取出。