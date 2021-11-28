# Spring AOP

## Spring AOP的作用

在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等待
* 就是把一些系统的业务比如日志管理，事务管理等变成横向切面，将其跟核心业务逻辑代码分开，使代码比较简洁。
* 等到要用到这个切面的时候，就把它织入到目标对象中成代理对象，实际上操作的是代理对象。 采用横向抽取机制，取代了传统纵向继承体系重复性代码。

```
你写了个方法用来做一些事情，但这个事情要求登录 用户才能做，你就可以在这个方法执行前验证一下，执行后记录下操作日志，把前后的这些与业务逻辑无关的代码抽取出来放一个类里，这个类就是切面（Aspect），这个被环绕的方法就是切点（Pointcut），你所做的执行前执行后的这些方法统一叫做增强处理（Advice
```

##  AOP原理

* Spring AOP通过Pointcut（切点）指定在哪些类的哪些方法上施加横切逻辑，
* 通过Advice（增强）描述横切逻辑和方法的具体织入点（方法前、方法后、方法的两端等），
* 把前后的这些与业务逻辑无关的代码抽取出来放一个类里,这个类就是切面（Aspect）
  * Spring还通过Advisor（切面）组合Pointcut和Advice。有了Advisor的信息，Spring就可以利用JDK或CGLib的动态代理技术为目标Bean创建织入切面的代理对象了


AOP底层使用动态代理实现


### AOP 两种代理机制

*  基于Jdk实现InvocationHandler 底层使用反射技术
*  基于CGLIB实现 字节码技术

Spring在选择用JDK还是CGLiB的依据：
1. 当Bean实现接口时，Spring就会用JDK的动态代理。JDK本身只提供基于接口的代理，不支持类的代理。
2. 当Bean没有实现接口时，Spring使用CGlib实现。cglib没有这样的要求
3. 可以强制使用CGlib（在spring配置中加入<aop:aspectj-autoproxy proxy-target-class=“true”/>）



**JDK**
* `java.lang.reflect`包中的两个类：`Proxy`和`InvocationHandler`
* 其中`InvocationHandler`是一个接口，可以通过实现该接口定义横切逻辑，在并通过反射机制调用目标类的代码，动态将横切逻辑和业务逻辑编织在一起。
* `Proxy`为`InvocationHandler`实现类动态创建一个符合某一接口的代理实例

```
TargetInterface proxy = (TargetInterface) Proxy.newProxyInstance(target.getClass.getClassLoader(), target.getClass.getInterfaces(), new LogHandler(new target()));
```
*  

1. 定义一个实现接口InvocationHandler的类
2. 通过构造函数，注入被代理类
3. 实现invoke（ Object proxy, Method method, Object[] args）方法, 实现功能上的增强的
4. 使用Proxy.newProxyInstance( )创建代理类并生成相应的代理对象；
5. 通过代理对象调用各种方法



CGlib: 采用非常底层的字节码技术，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑
* 通过字节码技术为一个类创建子类(继承的方式)，
* 并在子类中采用方法拦截的技术拦截所有父类方法的调用，
* 并在拦截方法相应地方织入横切逻辑

**CGLIB的核心类**
* `net.sf.cglib.proxy.Enhancer` – 主要的增强类
* `net.sf.cglib.proxy.MethodInterceptor` – 主要的方法拦截类，它是`Callback`接口的子接口，需要用户实现
* `net.sf.cglib.proxy.MethodProxy` – JDK的java.lang.reflect.Method类的代理类，可以方便的实现对源对象方法的调用,如使用：
 Object o = methodProxy.invokeSuper(proxy, args);//虽然第一个参数是被代理对象，也不会出现死循环的问题。


1. 定义一个实现了MethodInterceptor接口的类
2. 实现其intercept()方法，在其中调用proxy.invokeSuper( )

```
public class HelloServiceInterceptor implements MethodInterceptor{
 
    /**
     * sub：cglib生成的代理对象
     * method：被代理对象方法
     * objects：方法入参
     * methodProxy: 代理方法
     */
     
    @Override
    public Object intercept(Object sub, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("======插入前置通知======");
        Object object = methodProxy.invokeSuper(sub, objects);
        System.out.println("======插入后置通知======");
        return object;
}

Enhancer enhancer = new Enhancer();
// 设置enhancer对象的父类
enhancer.setSuperclass(HelloService.class);
// 设置enhancer的回调对象
enhancer.setCallback(new HelloServiceInterceptor());
// 创建代理对象
HelloService proxy= (HelloService)enhancer.create();
// 通过代理对象调用目标方法
proxy.sayHello();
        
```



**Summary**
* CGLib所创建的动态代理对象的性能依旧比JDK的所创建的代理对象的性能高不少（大概10倍）。
* CGLib在创建代理对象时性能却比JDK动态代理慢很多（大概8倍），
* 所以对于singleton的代理对象或者具有实例池的代理，因为不需要频繁创建代理对象，所以比较适合用CGLib动态代理技术，反之适合用JDK动态代理技术。
* 此外，由于CGLib采用生成子类的技术创建代理对象，所以不能对目标类中的final方法进行代理


> JDK跟CGLIB的区别，CGLIB可不可以代理JDK代理代理的类,cglib可以代理jdk代理的类吗

* jdk动态代理只可以代理接口，因为最后的实现类要继承Proxy并实现该接口
    *  它的实现原理是通过利用反射机制 InvocationHandler.invoke方法实现对实现类方法的调用（InvocationHandler实例已经持有了对实现类对象的引用了），然后实现方法前后的拦截
* CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法（继承）,然后通过MethodIntercept.intercept方法来实现在调用父类方法的前后执行拦截


## 代理模式

为其他对象提供一种代理以控制对这个对象的访问，在不修改被代理对象的基础上，通过扩展代理类，进行一些功能的附加与增强。值得注意的是，代理类和被代理类应该共同实现一个接口，或者是共同继承某个类。

1. 接口：原有方法实现了接口，我也写一个类实现接口，方法重写，在内容中回调原有方法，并在回调前后进行增强。
2. 继承：父类实现了方法，我编写一个子类重写父类的方法，同样在内容中回调原有方法（保证原有业务逻辑不变），并在回调前后进行增强。



##  静态代理和动态代理的区别：


静态代理
*  自己编写创建代理类，然后再进行编译，而且是在编译器就已经确定被代理的对象 在程序运行前，代理类的.class文件就已经存在了
*  代理使客户端不需要知道实现类是什么，怎么做的，而客户端只需知道代理即可,静态代理事先知道要代理的是什么，而动态代理不知道要代理什么东西
*  每个代理类只能为一个接口服务，这样程序开发中必然会产生许多的代理类

动态代理
* 动态代理是在运行时，通过反射机制实现动态代理，
* 并且能够代理各种类型的对象
* 可以通过一个代理类完成全部的代理功能,能够代理各种类型的对象
* 在Java中要想实现动态代理机制，需要java.lang.reflect.InvocationHandler接口和 java.lang.reflect.Proxy 类的支持
* 最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。而且动态代理的应用使我们的类职责更加单一，复用性更强


区别
* 静态代理在编译时就已经实现，编译完成后代理类是一个实际的class文件 ，动态代理是在运行时动态生成的，即编译完成后没有实际的class文件，而是在运行时动态生成类字节码，并加载到JVM中
* 静态代理通常只代理一个类，动态代理是代理一个接口下的多个实现类。
* 静态代理事先知道要代理的是什么，而动态代理不知道要代理什么东西，只有在运行时才知道。
* 最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。而且动态代理的应用使我们的类职责更加单一，复用性更强
