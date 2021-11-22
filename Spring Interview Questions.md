> Spring 事务失效的原因




> SpringBoot 自动配置原理

搭SpringBoot环境的时候,无须各种的配置文件，无须各种繁杂的pom坐标，一个main方法，就能run起来了

@SpringBootApplication等同于下面三个注解：

* @SpringBootConfiguration
* @EnableAutoConfiguration: 这个注解可以帮助我们自动载入应用程序所需要的所有默认配置
* @ComponentScan


其中@EnableAutoConfiguration 代表开启自动装配
* 注解会去 spring-boot-autoconfigure工程下寻找 META-INF/spring.factories 文件，此文件中列举了所有能够自动装配类的清单，然后自动读取里面的自动装配配置类清单。
* 将其文件包装成Properties对象, 从Properties对象获取到key值为EnableAutoConfiguration的数据，然后添加到容器里边加载到IOC容器中，实现自动配置功能
* 因为有 @ConditionalOn 条件注解，满足一定条件配置才会生效，否则不生效。 如：  @ConditionalOnClass(某类.class)  工程中必须包含一些相关的类时，配置才会生效。
* 所以说当我们的依赖中引入了一些对应的类之后，满足了自动装配的条件后，自动装配才会被触

<img width="816" alt="Screen Shot 2021-11-19 at 9 16 00 AM" src="https://user-images.githubusercontent.com/27160394/142528331-6de665cd-67c1-420a-bdc2-830217140465.png">


* @AutoConfigurationPackage  ////自动导包 
  * 本身的含义就是将主配置类（@SpringBootApplication 标注的类）所在的包下面所有的组件都扫描到 spring 容器中
* @Import(AutoConfigurationImportSelector.class) ////自动配置导入选择: 开启自动配置类的导包的选择器 
    * 将所有需要导入的组件以全类名的方式返回，并添加到容器中，最终会给容器中导入非常多的自动配置类（xxxAutoConfiguration），给容器中导入这个场景需要的所有组件，并配置好这些组件
    * 其导入的`AutoConfigurationImportSelector`的`selectImports()`方法通过`SpringFactoriesLoader.loadFactoryNames()`扫描所有具有META-INF/spring.factories的jar包下面key是EnableAutoConfiguration全名的，所有自动配置类
* 内部实际上就去加载META-INF/spring.factories文件的信息，然后筛选出以EnableAutoConfiguration为key的数据，加载到IOC容器中，实现自动配置功能！
* Spring启动的时候会扫描所有jar路径下的META-INF/spring.factories，将其文件包装成Properties对象
* 从Properties对象获取到key值为EnableAutoConfiguration的数据，然后添加到容器里边



根据spring.factories配置加载EnableAutoConfiguration
其中给容器中自动配置添加组件的时候，会从propeties类中获取配置文件中指定这些属性的值。xxxAutoConfiguration：⾃动配置类给容器中添加组件。xxxProperties：封装配置⽂件中相关属性。
根据@Conditional注解的条件，进行自动配置并将Bean注入Spring容器



> Spring的三级缓存

Spring内部维护了三个Map，也就是我们通常说的三级缓存

三级缓存分别是：
* singletonObjects： 一级缓存，存储单例对象，Bean 已经实例化，初始化完成。存放完全实例化属性赋值完成的Bean，直接可以使用，成品Bean
* earlySingletonObjects： 二级缓存，存储 singletonObject，这个 Bean 实例化了，还没有初始化。存放早期Bean的引用，尚未属性装配的Bean，半成品的Bean
* singletonFactories： 三级缓存，存储 singletonFactory，三级缓存，存放实例化完成的Bean工厂。此缓存存的bean 工厂对象，也就存的是 专门创建Bean的一个工厂对象。此缓存用于解决循环依赖


> Spring 怎么解决依赖循环

单例的setter注入

Spring内部维护了三个Map，也就是我们通常说的三级
Spring解决循环依赖的核心思想在于提前曝光

Spring通过三级缓存解决了循环依赖，其中一级缓存为单例池（singletonObjects）,二级缓存为早期曝光对象earlySingletonObjects，三级缓存为早期曝光对象工厂（singletonFactories）。
* A引用创建后，提前暴露到半成品缓存中
* 依赖B，创建B ，B填充属性时发现依赖A， 先从成品缓存查找，没有,再从半成品缓存查找 取到A的早期引用。
* B顺利走完创建过程, 将B的早期引用从半成品缓存移动到成品缓存
* B创建完成，A获取到B的引用，继续创建。
* A创建完成，将A的早期引用从半成品缓存移动到成品缓存
* 完美解决循环依赖


> 为啥需要三个缓存

```
严格来讲，第三级缓存并非缺它不可，因为可以提前创建代理对象
Spring 的设计原则是尽可能保证普通对象创建完成之后，再生成其 AOP 代理（尽可能延迟代理对象的生成）
所以 Spring 用了第三级缓存，既维持了设计原则，又处理了循环依赖
```

如果创建的Bean有对应的代理，那其他对象注入时，注入的应该是对应的代理对象；但是Spring无法提前知道这个对象是不是有循环依赖的情况，而正常情况下(没有循环依赖情况）Spring都是在创建好完成品Bean之后才创建对应的代理。
* 不提前创建好代理对象，在出现循环依赖被其他对象注入时，才实时生成代理对象。这样在没有循环依赖的情况下，Bean就可以按着Spring设计原则的步骤来创建
* Spring就是在对象外面包一层ObjectFactory，提前曝光的是ObjectFactory对象，在被注入时才在ObjectFactory.getObject方式内实时生成代理对象，并将生成好的代理对象放入到第二级缓存


如果要使用二级缓存解决循环依赖，意味着所有Bean在实例化后就要完成AOP代理，这样违背了Spring设计的原则，Spring在设计之初就是通过AnnotationAwareAspectJAutoProxyCreator这个后置处理器来在Bean生命周期的最后一步来完成AOP代理，而不是在实例化后就立马进行AOP代理



考虑代理的情况。
* 代理的存在,Bean在创建的最后阶段，会检查是否需要创建代理，如果创建了代理，那么最终返回的就是代理实例的引用。我们通过beanname获取到最终是代理实例的引用

当A、B两个类发生循环引用时，在A完成实例化后，就使用实例化后的对象去创建一个对象工厂，并添加到三级缓存中，如果A被AOP代理，那么通过这个工厂获取到的就是A代理后的对象，如果A没有被AOP代理，那么这个工厂获取到的就是A实例化的对象

* A首先完成了初始化的第一步，并且将自己提前曝光到singletonFactories中，
* 此时进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走create流程，
* B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存singletonObjects(肯定没有，因为A还没初始化完全)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，
   * 由于A通过ObjectFactory将自己提前曝光了，所以B能够通过ObjectFactory.getObject拿到A对象(虽然A还没有初始化完全，但是总比没有好呀)，
* B拿到A对象后顺利完成了初始化阶段1、2、3，完全初始化之后将自己放入到一级缓存singletonObjects中。
* 此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，进去了一级缓存singletonObjects中，
* 而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象完成了初始化。



> 为啥Spring不能解决“A的构造方法中依赖了B的实例对象，同时B的构造方法中依赖了A的实例对象”这类问题了
* 构造方法注入的方式，将实例化与初始化并在一起完成，能够快速创建一个可直接使用的对象，但它没法处理循环依赖的问题，了解就好
* setter 方法注入的方式，是在对象实例化完成之后，再通过反射调用对象的 setter 方法完成属性的赋值，能够处理循环依赖的问题，
因为加入singletonFactories三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决。



> Bean 加载机制


> Bean的生命周期

Spring 创建Bean的过程，大致和对象的初始化有点类似吧。有几个关键的步骤
1. createBeanInstance ：实例化，此处要强调的是，Bean的早期引用在此出现了。
2. populateBean ： 填充属性，此处我们熟悉的@Autowired属性注入就发生在此处
3. initializeBean : 调用一些初始化方法，例如init ,afterPropertiesSet

此外：BeanPostProcessor作为一个扩展接口，会穿插在Bean的创建流程中，留下很多钩子，让我们可以去影响Bean的创建过程。其中最主要的就属AOP代理的创建了


> SpringBoot 如何固定版本


> SpringBoot 配置文件注入

> Spring AOP


在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等待
就是把一些系统的业务比如日志管理，事务管理等变成横向切面，将其跟核心业务逻辑代码分开，使代码比较简洁。等到要用到这个切面的时候，就把它织入到目标对象中成代理对象，实际上操作的是代理对象。 采用横向抽取机制，取代了传统纵向继承体系重复性代码。

将日志记录，性能统计，安全控制，事务处理，异常处理等代码从业务逻辑代码中划分出来，通过对这些行为的分离，我们希望可以将它们独立到非指导业务逻辑的方法中，进而改变这些行为的时候不影响业务逻辑的代码


你写了个方法用来做一些事情，但这个事情要求登录用户才能做，你就可以在这个方法执行前验证一下，执行后记录下操作日志，把前后的这些与业务逻辑无关的代码抽取出来放一个类里，这个类就是切面（Aspect），这个被环绕的方法就是切点（Pointcut），你所做的执行前执行后的这些方法统一叫做增强处理（Advice


AOP原理

动态代理技术，综合运用两种代理模式
基于Jdk实现InvocationHandler 底层使用反射技术
基于CGLIB实现 字节码技术
创建代理时，如果有接口则执行jdk动态代理，否则执行cglib动态代理。

利用@EnableAspectJAutoProxy注解增强

为其他对象提供一种代理以控制对这个对象的访问，在不修改被代理对象的基础上，通过扩展代理类，进行一些功能的附加与增强。值得注意的是，代理类和被代理类应该共同实现一个接口，或者是共同继承某个类。

自己手写代理类就是静态代理。手写代理类也有两种思路，一是通过继承被代理类的方式实现其子，重写父类方法；二是与被代理类实现共同的一个接口，尴尬的一点是被代理类未必会有接口。
动态代理就是交给程序去自动生成代理类，即基于接口的JDK动态代理和基于继承的cglib动态代理。

JDK
1.定义一个实现接口InvocationHandler的类
2. 通过构造函数，注入被代理类
3. 实现invoke（ Object proxy, Method method, Object[] args）方法
4. 使用Proxy.newProxyInstance( )产生一个代理对象
5. 通过代理对象调用各种方法

1.创建被代理的接口和类；
2. 实现InvocationHandler接口，对目标接口中声明的所有方法进行统一处理；
3. 调用Proxy的静态方法，创建代理类并生成相应的代理对象；
4. 通过代理对象调用各种方法

CGlib
1. 定义一个实现了MethodInterceptor接口的类
2. 实现其intercept()方法，在其中调用proxy.invokeSuper( )

> 静态代理和动态代理的区别：

静态代理：自己编写创建代理类，然后再进行编译，而且是在编译器就已经确定被代理的对象 在程序运行前，代理类的.class文件就已经存在了。 
代理使客户端不需要知道实现类是什么，怎么做的，而客户端只需知道代理即可,
每个代理类只能为一个接口服务，这样程序开发中必然会产生许多的代理类


所以我们就会想办法可以通过一个代理类完成全部的代理功能，那么我们就需要用动态代理
动态代理：动态代理是在运行时，通过反射机制实现动态代理，并且能够代理各种类型的对象
在Java中要想实现动态代理机制，需要java.lang.reflect.InvocationHandler接口和 java.lang.reflect.Proxy 类的支持

在实现阶段不用关心代理谁，而在运行阶段（通过反射机制）才指定代理哪一个对象。


动态代理与静态代理相比较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。而且动态代理的应用使我们的类职责更加单一，复用性更强




> Spring IOC

这个容器说白了就是把你放在里面的对象（Bean）进行统一管理，你不用考虑对象如何创建如何销毁，从这方面来说，所谓的控制反转就是获取对象的方式被反转了。既然你都把对象交给人家Spring管理了，那你需要的时候不得给人家要呀。这就是依赖注入（DI）！再想下，我们在传入一个参数的时候除了在构造方法中就是在setter方法中，换个好听的名字就是构造注入和设值注入


某一个接口具体实现类的选择控制权从调用类中移除，转交给第三方决定，在 Spring 容器中是由 Bean 配置来进行控制的

* 对于Spring，核心就是IOC容器，这个容器说白了就是把你放在里面的对象（Bean）进行统一管理，你不用考虑对象如何创建如何销毁，从这方面来说，所谓的控制反转就是获取对象的方式被反转了。既然你都把对象交给人家Spring管理了，那你需要的时候不得给人家要呀。这就是依赖注入（DI）

在传统的程序设计中，当调用者需要被调用者的协助时，通常由调用者来创建被调用者的实例。但在spring里创建被调用者的工作不再由调用者来完成，因此控制反转（IoC）；创建被调用者实例的工作通常由spring容器来完成，然后注入调用者，因此也被称为依赖注入（DI），依赖注入和控制反转是同一个概念。

Spring IoC 的底层实现是基于反射技术
* 使用反射技术获取对象的信息包括：类信息、成员、方法等等
* 再通过 xml 配置 或者 注解 的方式，说明依赖关系
* 在调用类需要使用其他类的时候，不再通过调用类自己实现，而是通过 IoC 容器进行注入。


> MVC中，@RequestMapping的实现原理？这边没有了解过，询问了你来设计会怎么设计？url与接口的怎么完成注册？怎么根据url匹配到接口？如果匹配到多个接口，如果选择？
 
> Spring中的ioc和aop，ioc的注解有哪些，autowired和resource有什么区别，作用域有哪些，autowired如何配置两个类中的一个。
 
> spring生命周期，几种scope区别，aop实现有哪几种实现，接口代理和类代理会有什么区别
 

> @Value 和 @ConfigurationProperties 比较
> @PropertySource
> @ImportResource
> springboot 的 profile 加载
> SpringBoot **指定 profile 的几种方式
> SpringBoot 项目内部配置文件加载顺序
>  SpringBoot 外部配置文件加载顺序
>  Springboot 日志关系
>  SpringBoot 如何扩展 SpringMVC 的配置
>  SpringBoot 如何注册 filter ， servlet ， listener
>  SpringBoot 切换成 undertow
>  SpringBoot 的任务
>  SpringBoot 热部署
