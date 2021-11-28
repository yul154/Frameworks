> Spring 事务失效的原因

> SpringBoot的四个特点
1. 更快速的构建能力 -> 提供了更多的 Starters 用于快速构建业务框架
2. 起步依赖 -> 创建 Spring Boot 时可以直接勾选依赖模块，这样在项目初始化时就会把相关依赖直接添加到项目中，大大缩短了查询并添加依赖的时间

> spring boot 启动过程

Spring Boot的启动过程大致可以分为如下2步
1. 初始化一个SpringApplication对象  -> 为运行SpringApplication实例对象启动环境变量准备以及进行必要的资源构造器的初始化动作
  * 通过SpringFactoriesLoader找到`spring.factories`文件中配置的`ApplicationContextInitializer``ApplicationListener`
  * 配置基本的环境变量、资源、构造器和监听器
2. 执行该对象的run方法,
 1. 通过SpringFactoriesLoader查找并加载所有的,SpringApplicationRunListeners 引用启动监控模块
 2.  ConfigurableEnvironment, 创建并配置当前应用将要使用的Environment
 3. ConfigrableApplicationContext配置应用上下文：包括配置应用上下文对象、配置基本属性和刷新应用上下文  


> Spring 起步依賴

起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。很多起步依赖的命名都暗示了它们提供的某种或某类功能

Spring Boot起步依赖大大简化了项目构建说明中的依赖配置，因为常用的依赖聚合于更粗粒度的依赖。你的构建项目会传递解析到起步依赖中声明的其他依赖。

> SpringBoot 自动配置原理/过程

1. springboot启动时，是依靠启动类的main方法来进行启动的，而main方法中执行的是SpringApplication.run()方法
2. `SpringApplication.run()`方法中会创建spring的容器，并且刷新容器。而在刷新容器的时候就会去解析启动类，然后就会去解析启动类上的`@SpringBootApplication`注解
3. `@SpringBootApplication`注解是个复合注解
   * @SpringBootConfiguration
   * @EnableAutoConfiguration: 这个注解就是开启自动配置
   * @ComponentScan
4.`@EnableAutoConfiguration`注解中又有`@Import`注解引入了一个`AutoConfigurationImportSelector`这个类，这个类会进过一些核心方法，
5. 然后去扫描我们所有jar包下的META-INF下的spring.factories文件,在这个文件中记录了好多的自动配置类
6. 而从这个配置文件中取找key为EnableAutoConfiguration类的全路径的值下面的所有配置都加载

```
SpringBoot在启动的时候会调用run()方法，run()方法会刷新容器，刷新容器的时候，会扫描classpath下面的的包中META-INF/spring.factories文件，
在这个文件中记录了好多的自动配置类，在刷新容器的时候会将这些自动配置类加载到容器中，然后在根据这些配置类中的条件注解，来判断是否将这些配置类在容器中进行实例化，
这些条件主要是判断项目是否有相关jar包或是否引入了相关的bean。这样springboot就帮助我们完成了自动装配
```

其中@EnableAutoConfiguration 代表开启自动装配
* 注解会去 spring-boot-autoconfigure工程下寻找 META-INF/spring.factories 文件，此文件中列举了所有能够自动装配类的清单，然后自动读取里面的自动装配配置类清单。
* 将其文件包装成Properties对象, 从Properties对象获取到key值为EnableAutoConfiguration的数据，然后添加到容器里边加载到IOC容器中，实现自动配置功能
* 因为有 @ConditionalOn 条件注解，满足一定条件配置才会生效，否则不生效。 如：  @ConditionalOnClass(某类.class)  工程中必须包含一些相关的类时，配置才会生效。
* 所以说当我们的依赖中引入了一些对应的类之后，满足了自动装配的条件后，自动装配才会被触

<img width="816" alt="Screen Shot 2021-11-19 at 9 16 00 AM" src="https://user-images.githubusercontent.com/27160394/142528331-6de665cd-67c1-420a-bdc2-830217140465.png">


* @AutoConfigurationPackage  ////自动导包 
  * 本身的含义就是将主配置类（@SpringBootApplication 标注的类）所在的包下面所有的组件都扫描到 spring 容器中
* @Import(AutoConfigurationImportSelector.class) //自动配置导入选择: 开启自动配置类的导包的选择器 
    * 将所有需要导入的组件以全类名的方式返回，并添加到容器中，最终会给容器中导入非常多的自动配置类（xxxAutoConfiguration），给容器中导入这个场景需要的所有组件，并配置好这些组件
    * 其导入的`AutoConfigurationImportSelector`的`selectImports()`方法通过`SpringFactoriesLoader.loadFactoryNames()`扫描所有所有jar包的META-INF/spring.factories
    * 将这些扫描到的文件转成Properties对象
    * 只获取了key是`EnableAutoConfiguration.class`的所有自动配置类

> 配置加载顺序
spring boot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

1. 类路径下的配置文件
2. 类路径内config子目录的配置文件
3. 当前项目根目录下的配置文件
4. 当前项目根目录下config子目录的配置文件

SpringBoot配置文件存在一个特性，优先级较高的配置加载顺序比较靠后，相同名称的配置优先级较高的会覆盖掉优先级较低的内容


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



> Bean 的装配过程：
* BeanDefinitionReader 读取 Resource 所指向的配置文件资源，然后解析配置文件。配置文件中每一个解析成一个 BeanDefinition 对象，并保存到 BeanDefinitionRegistry 中；
* 容器扫描 BeanDefinitionRegistry 中的BeanDefinition；调用InstantiationStrategy 进行Bean实例化的工作；使用 BeanWrapper 完成Bean属性的设置工作；
* 单例Bean缓存池：Spring 在DefaultSingletonBeanRegistry类中提供了一个用于缓存单实例 Bean 的缓存器，它是一个用HashMap实现的缓存器，单实例的 Bean 以beanName为键保存在这个HashMap中。


> Bean的生命周期

Spring 创建Bean的过程，大致和对象的初始化有点类似吧。有几个关键的步骤
1. createBeanInstance ：实例化，此处要强调的是，Bean的早期引用在此出现了。
2. populateBean ： 填充属性，此处我们熟悉的@Autowired属性注入就发生在此处
3. initializeBean : 调用一些初始化方法，例如init ,afterPropertiesSet

此外：BeanPostProcessor作为一个扩展接口，会穿插在Bean的创建流程中，留下很多钩子，让我们可以去影响Bean的创建过程。其中最主要的就属AOP代理的创建了

> Bean 作用域
* singleton：单例，只有一个对象
* prototype：多例，每次都会创建一个新对象
* request：每次请求都会参数一个对象
* session：每个session都会产生一个对象
* global session：所有的session都用一个对象

每接收到一个Http请求时，将HttpServletRequest和HttpServletResponse封装成一个ServletRequestAttributes 绑定到RequestContextHolder的ThreadLocal变量中


## Bean 生命周期
1. 实例化（Instantiation）；
2.属性赋值（Populate）；
3. 初始化（Initialization）；
4. 销毁（Destruction）。

---
## Spring 主要思想

* IOC(DI): Inversion of Control (控制反转/反转控制)，指的是关于对象的创建和管理的控制权反转了;依赖别人者不会因被依赖者改变而改变，达到了高度的松耦合
* AOP:把前后的这些与业务逻辑无关的代码剝離，实现原有方法业务内容不变，前后进行增强的模式


### AOP
> 描述一下Spring AOP？

> 在Spring AOP中关注点(concern)和横切关注点(cross-cutting concern)有什么不同？

> AOP有哪些可用的实现？

> Spring中有哪些不同的通知类型(advice types)？

> Spring AOP 代理是什么？

> 引介(Introduction)是什么？

> 连接点(Joint Point)和切入点(Point Cut)是什么？

> 织入（Weaving）是什么？


----
> Spring 究竟是如何知道哪些对象是需要管理的呢？如何进行管理的呢？又是如何进行注入的呢？

* Spring 通过一个配置文件或注解来描述 Bean 和 Bean 之间的依赖关系
* 根据Bean配置信息在容器内部创建Bean定义注册表，根据注册表加载、实例化 Bean、建立Bean与Bean之间的依赖关系，
* 还提供了 Bean 实例缓存、生命周期管理、Bean 实例代理、事件发布、资源装载等高级服务

> Spring中的ioc和aop，ioc的注解有哪些，autowired和resource有什么区别，作用域有哪些，autowired如何配置两个类中的一个。


> BeanFactory 和 ApplicationContext 有什么区别

* BeanFactory 可以理解为含有 bean 集合的工厂类。BeanFactory 包含了种 bean 的定义，以便在接收到客户端请求时将对应的bean 实例化。
  * BeanFactory 还能在实例化对象的时生成协作类之间的关系。此举将 bean 自身与 bean 客户端的配置中解放出来。BeanFactory 还包含了 bean 生命周期的控制，调用客户端的初始化方法（initialization methods）和销毁方法（destruction methods）。

* 从表面上看，application context 如同 bean factory 一样具有 bean 定义、bean 关联关系的设置，根据请求分发bean 的功能。但 application context 在此基础上还提供了其他的功能。
 * 提供了支持国际化的文本消息
 * 统一的资源文件读取方式
 *  已在监听器中注册的 bean 的事件

* BeanFactory 在启动的时候不会去实例化 Bean，中有从容器中拿 Bean 的时候才会去实例化，ApplicationContext 在启动的时候就把所有的 Bean 全部实例化了。它还可以为 Bean 配置 lazy-init=true 来让 Bean 延迟实例化；

* BeanFacotry 是 spring 中比较原始的 Factory，无法支持 spring 的许多插件，如 AOP 功能、Web 应用等。 ApplicationContext接口



> MVC中，@RequestMapping的实现原理？这边没有了解过，询问了你来设计会怎么设计？url与接口的怎么完成注册？怎么根据url匹配到接口？如果匹配到多个接口，如果选择？
 
 
> spring生命周期，几种scope区别，aop实现有哪几种实现，接口代理和类代理会有什么区别
 
> springboot加载过程，依赖于自动加载

> springboot 怎么实现依赖起步
起步依赖就是特殊的Maven项目模型.定义了一下其他库的传递依赖，这些依赖加起来支撑着某项功能
* 利用了传递依赖解析，把常用库聚合在一起，组成几个为特定功能而定制的依赖。
* Spring Boot通过起步依赖：直接引入相关起步依赖就行，我们不需要考虑支持某种功能需要什么库,减少了依赖数量，而且不需要考虑这些库的那些版本。如果我们需要什么功能，就往项目中加入该功能的起步依赖就好了

 
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
