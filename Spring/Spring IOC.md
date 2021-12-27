# Spring IOC
- IOC 原理
- DI
  - 自动装配
- IOC 实现原理
 - 容器框架
 - 容器初始化
 - bean初始化

## Spring IOC的原理

```
* 对于Spring，核心就是IOC容器，这个容器说白了就是把你放在里面的对象（Bean）进行统一管理，你不用考虑对象如何创建如何销毁，从这方面来说，所谓的控制反转就是获取对象的方式被反转了。既然你都把对象交给人家Spring管理了，那你需要的时候不得给人家要呀。这就是依赖注入（DI）
* 在传统的程序设计中，当调用者需要被调用者的协助时，通常由调用者来创建被调用者的实例。
* 但在spring里创建被调用者的工作不再由调用者来完成，因此控制反转（IoC）；创建被调用者实例的工作通常由spring容器来完成，然后注入调用者，因此也被称为依赖注入（DI），
````

IoC 全称为 InversionofControl，翻译为 “控制反转”.
* 面向对象编程中的一种设计原则，一种设计思想 可以用来减低计算机代码之间的耦合度
* Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制

**谁控制谁** 
* 在传统的开发模式下,我们都是采用直接`new`一个对象的方式来创建对象,也就是说你依赖的对象直接由你自己控制
* 但是有了 IOC 容器后，则直接由IoC容器来控制。IoC 容器控制对象

**为何是反转，哪些方面反转了**
* 正转: 没有IoC的时候我们都是在自己对象中主动去创建被依赖的对象
* 反转 但是有了IoC后，所依赖的对象直接由IoC容器创建后注入到被注入的对象中，依赖的对象由原来的主动获取变成被动接受，所以是反转.
* 依赖对象的获取方面反转了


## IoC能做什么
> IoC 不是一种技术，只是一种思想

把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是 松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活。

IoC很好的体现了面向对象设计法则之一—— 好莱坞法则：“别找我们，我们找你”


## Ioc 配置的三种方式


**1.xml 配置**

将bean的信息配置.xml文件里，通过Spring加载文件为我们创建bean

**2.Java 配置**

将类的创建交给我们配置的JavcConfig类来完成，Spring只负责维护和管理，采用纯Java创建方式。其本质上就是把在XML上的配置声明转移到Java配置类中


**3.注解配置**

通过在类上加注解的方式，来声明一个类交给Spring管理，Spring会自动扫描带有@Component，@Controller，@Service，@Repository这四个注解的类，然后帮我们创建并管理，前提是需要先配置Spring的注解扫描器

1. 对类添加@Component相关的注解，比如@Controller，@Service，@Repository 
2. 设置ComponentScan的basePackage,`@ComponentScan("tech.pdai.springframework")`注解，或者 `new AnnotationConfigApplicationContext("tech.pdai.springframework"`)指定扫描的basePackage.

-----
# DI
> 控制反转是通过依赖注入实现的，其实它们是同一个概念的不同角度描述。通俗来说就是IoC是设计思想，DI是实现方式


DI的关键是：“谁依赖谁，为什么需要依赖，谁注入谁，注入了什么
* 被注入的对象”依赖“依赖对象
* 容器管理对象需要IoC容器来提供对象需要的外部资源
* 很明显是IoC容器注入某个对象，也就是注入“依赖对象”
* 就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）


##   依赖注入方式
对于spring配置一个bean时，如果需要给该bean提供一些初始化参数，则需要通过依赖注入方式
* 依赖注入就是通过spring将bean所需要的一些参数传递到bean实例对象的过程

在创建新的Bean时，IoC容器会自动注入新Bean的所依赖的其他Bean

1. 构造器注入
2. setter 方法注入
3. 字段注入

```
private DependencyA dependencyA;
public DI(DependencyA dependencyA,)

@Autowired
public void setDependencyA(DependencyA dependencyA) {


@Autowired
private DependencyA dependencyA;
```

> 为什么推荐构造器注入方式？

* 能够保证注入的组件不可变 : 其实说的就是final关键字
* 并且确保需要的依赖不为空 :由于自己实现了有参数的构造函数，所以不会调用默认构造函数，那么就需要Spring容器传入所需要的参数，所以就两种情况：1、有该类型的参数->传入，OK 2：无该类型的参数->报错。
* 构造器注入的依赖总是能够在返回客户端（组件）代码的时候保证完全初始化的状态。 在Java类加载实例化的过程中，构造方法是最后一步，所以返回来的都是初始化之后的状态

```
public UserServiceImpl(final UserDaoImpl userDaoImpl) {
        this.userDao = userDaoImpl;
    }
```

“装配”是把类转为由spring容器管理的bean。 “注入”是把spring容器管理的bean赋值给类中的成员变量，即属性。

### 装配
> 依赖注入的本质就是装配，装配是依赖注入的具体行为

当一个对象的属性是另一个对象时，实例化时，需要为这个对象属性进行实例化。这就是装配。
* Spring 装配包括手动装配和自动装配，手动装配是有基于xml装配、 构造方法、 setter 方法等；
* 自动装配其实是依赖注入的升级版，为了简化依赖注入的配置而生成的
* `@ComponentScan("")`

自动装配有五种自动装配的方式，可以用来指导Spring容器用自动装配方式来进行依赖注入。
```
1. no：默认的方式是不进行自动装配，通过显式设置 ref 属性来进行装配。
2. byName：通过参数名 自动装配， Spring 容器在配置文件中发现 bean 的 autowire 属性被设置成 byname，之后容器试图匹配、装配和该 bean 的属性具有相同名字的 bean。
3. byType：通过参数类型自动装配， Spring 容器在配置文件中发现 bean 的 autowire 属性被设置成 byType，之后容器试图匹配、装配和该 bean 的属性具有相同类型的 bean。如果有多个 bean 符合条件，则抛出错误。
4. constructor：这个方式类似于 byType， 但是要提供给构造器参数，如果没有确定的带参数的构造器参数类型，将会抛出异常。
5. autodetect：首先尝试使用 constructor 来自动装配，如果无法工作，则使用 byType 方式。
```



### 常用的自动装配注解有以下几种

@Autowired
* @Autowired是Spring自带的注解，通过`AutowiredAnnotationBeanPostProcessor`类实现的依赖注入
* @Autowired可以作用在CONSTRUCTOR、METHOD、PARAMETER、FIELD、ANNOTATION_TYPE
* @Autowired默认是根据类型（byType ）进行自动装配的
* 这个属性是强制性的，也就是说必须得装配上，如果没有找到合适的bean能够装配上，就会抛出异常，如果required=false时，则不会抛出异常。

@Qualifier: 
* @Resource是JSR250规范的实现，在javax.annotation包下
* @Resource可以作用TYPE、FIELD、METHOD上
* @Resource是默认根据属性名称进行自动装配的，如果有多个类型一样的Bean候选者，则可以通过name进行指定进行注入

@Inject
* @Inject是JSR330 (Dependency Injection for Java)中的规范，需要导入javax.inject.Inject jar包 ，才能实现注入
* @Inject可以作用CONSTRUCTOR、METHOD、FIELD上 
* @Inject是根据类型进行自动装配的，如果需要按名称进行装配，则需要配合@Named；
* @Autowired、@Inject用法基本一样，不同的是@Inject没有一个request属性


**Bean 的装配过程**
1. BeanDefinitionReader 读取 Resource 所指向的配置文件资源，然后解析配置文件。
2. 配置文件中每一个解析成一个 BeanDefinition 对象，并保存到 BeanDefinitionRegistry 中；
3. 容器扫描 BeanDefinitionRegistry 中的BeanDefinition；调用InstantiationStrategy 进行Bean实例化的工作；
4. 使用 BeanWrapper 完成Bean属性的设置工作；
5. 单例Bean缓存池：Spring 在DefaultSingletonBeanRegistry类中提供了一个用于缓存单实例 Bean 的缓存器，它是一个用HashMap实现的缓存器，单实例的 Bean 以beanName为键保存在这个HashMap中。


-----


# IOC的底层实现

<img width="532" alt="Screen Shot 2021-12-26 at 7 27 20 PM" src="https://user-images.githubusercontent.com/27160394/147406550-436e7e0d-b72a-417e-9569-b4df109f6515.png">

 ```
Spring IoC 的底层实现是基于反射技术
* 工厂+反射+配置文件(bean)
* 使用反射技术获取对象的信息包括：类信息、成员、方法等等
* 再通过 xml 配置 或者 注解 的方式，说明依赖关系
* 在调用类需要使用其他类的时候，不再通过调用类自己实现，而是通过 IoC 容器进行注入。
```

## IOC容器的主动功能

* 加载Bean的配置（比如xml配置）
  * 比如不同类型资源的加载，解析成生成统一Bean的定义 
* 根据Bean的定义加载生成Bean的实例，并放置在Bean容器中
  * 比如Bean的依赖注入，Bean的嵌套，Bean存放（缓存）等 
* 除了基础Bean外，还有常规针对企业级业务的特别Bean
  * 比如国际化Message，事件Event等生成特殊的类结构去支撑 
* 对容器中的Bean提供统一的管理和调用
  * 比如用工厂模式管理，提供方法根据名字/类的类型等从容器中获取Bean



## BeanFactory和BeanRegistry：IOC容器功能规范和Bean的注册

Spring IOC容器是怎么实现对象的创建和依赖的
1. 根据Bean配置信息在容器内部创建Bean定义注册表
2. 根据注册表加载、实例化bean、建立Bean与Bean之间的依赖关系
3. 将这些准备就绪的Bean放到Map缓存池中，等待应用程序调用



* BeanFactory : 工厂模式定义了IOC容器的基本功能规范
* BeanRegistry： 向IOC容器手工注册 BeanDefinition 对象的方法


### BeanFactory

BeanFactory作为最顶层的一个接口类，它定义了IOC容器的基本功能规范

```
public interface BeanFactory{
        //根据bean的名字和Class类型等来得到bean实例    
    Object getBean(String name) throws BeansException;  
    //返回指定bean的Provider
    <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
     //检查工厂中是否包含给定name的bean，或者外部注册的bean
    boolean containsBean(String name);
     //检查所给定name的bean是否为单例/原型
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException
    
/判断所给name的类型与type是否匹配
    boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
    
 //获取给定name的bean的类型
    @Nullable
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;
    

}

```

### BeanRegistry
> 将Bean注册到BeanFactory中

* Spring配置文件中每一个<bean>节点元素在 Spring 容器里都通过一个`BeanDefinition`对象表示，它描述了 Bean 的配置信息。
* `BeanDefinitionRegistry`接口提供了向容器手工注册 BeanDefinition 对象的方法

#### BeanDefinition
> 各种Bean对象及其相互的关系

* BeanDefinition 定义了各种Bean对象及其相互的关系 
* BeanDefinitionReader 这是BeanDefinition的解析器 
* BeanDefinitionHolder 这是BeanDefination的包装类，用来存储BeanDefinition，name以及aliases等。


### ApplicationContext：IOC接口设计和实现
> IoC容器的接口类是ApplicationContext，很显然它必然继承BeanFactory对Bean规范（最基本的ioc容器的实现）进行定义
        
ApplicationContext表示的是应用的上下文，除了对Bean的管理外，还至少应该包含了
* 访问资源： 对不同方式的Bean配置（即资源）进行加载。(实现ResourcePatternResolver接口) 
* 国际化: 支持信息源，可以实现国际化。（实现MessageSource接口） ]
* 应用事件: 支持应用事件。(实现ApplicationEventPublisher接口)
           

## IOC容器初始化的基本步骤
> Spring如何实现将资源配置（以xml配置为例）通过加载，解析，生成BeanDefination并注册到IoC容器中的
  
          
### 初始化的主体流程
        
Spring IoC容器对Bean定义资源的载入是从refresh()函数开始
* refresh()是一个模板方法
* refresh()方法的作用是：对IoC容器的重启，在新建立好的容器中对容器进行初始化，对Bean定义资源进行载入
        
![Screen Shot 2021-12-26 at 7 39 58 PM](https://user-images.githubusercontent.com/27160394/147406850-43846aa1-e3d8-4ae7-b300-03f0ae60a4bb.png)

        
* 模板方法中使用典型的钩子方法 将具体的初始化加载方法插入到钩子方法之间 
* 将初始化的阶段封装，用来记录当前初始化到什么阶段；常见的设计是xxxPhase/xxxStage； 
* 资源加载初始化有失败等处理，必然是try/catch/finally...
        
<img width="586" alt="Screen Shot 2021-12-26 at 7 29 49 PM" src="https://user-images.githubusercontent.com/27160394/147406606-aa42ea55-a5c1-497e-8c6a-910dc76a702e.png">

               
1. 初始化的入口在容器实现中的 refresh()调用来完成
2. 对 bean 定义载入 IOC 容器使用的方法是 loadBeanDefinition
   1. 通过 ResourceLoader来完成资源文件位置的定位,获得Resource对象。 DefaultResourceLoader 是默认的实现，同时上下文本身就给出了 ResourceLoader 的实现，可以从类路径，文件系统, URL 等方式来定为资源位置   
   2. BeanDefinitionReader读取Resource对象, 通过 BeanDefinitionReader来完成定义信息的解析和 Bean 信息的注册(), 往往使用的是XmlBeanDefinitionReader 来解析 bean 的 xml 定义文件
   3. 容器解析得到 BeanDefinition 以后，需要把它在 IOC 容器中注册，这由 IOC 实现 BeanDefinitionRegistry 接口来实现(BeanDefinition是容器内部Bean的基本数据结构，BeanFactory维持着一个BeanDefinition Map)
  
  

  
  
## Bean实例化 
> 如何从BeanDefinition中实例化Bean对象 
  
最终的将Bean的定义即BeanDefinition放到beanDefinitionMap中，本质上是一个ConcurrentHashMap<String, Object>；并且BeanDefinition接口中包含了这个类的Class信息以及是否是单例等；

       
**Spring什么时候实例化bean**
```
1. 如果你使用`BeanFactory`作为Spring Bean的工厂类，则所有的bean都是在第一次使用该Bean的时候实例化
2.如果你使用ApplicationContext作为Spring Bean的工厂类，则又分为以下几种情况：
   * 如果bean的scope是singleton的，并且lazy-init为false（默认是false，所以可以不用设置），则ApplicationContext启动的时候就实例化该Bean，并且将实例化的Bean放在一个map结构的缓存中，下次再使用该Bean的时候，直接从这个缓存中取 
   * 如果bean的scope是singleton的，并且lazy-init为true，则该Bean的实例化是在第一次使用该Bean的时候进行实例化 
   * 如果bean的scope是prototype的，则该Bean的实例化是在第一次使用该Bean的时候进行
``` 
        
### `getBean`
1. 从beanDefinitionMap通过beanName获得BeanDefinition 
2. 从BeanDefinition中获得beanClassName 
3. 通过反射初始化beanClassName的实例instance
    * 构造函数从BeanDefinition的getConstructorArgumentValues()方法获取 
    * 属性值从BeanDefinition的getPropertyValues()方法获取 
4. 返回beanName的实例instance

### Spring如何解决循环依赖问题
> Spring只是解决了单例模式下属性依赖的循环问题；Spring为了解决单例的循环依赖问题，使用了三级缓存
  
* 第一层缓存（singletonObjects）：单例对象缓存池，已经实例化并且属性赋值，这里的对象是成熟对象； 
* 第二层缓存（earlySingletonObjects）：单例对象缓存池，已经实例化并且属性赋值，这里的对象是半成品对象； 
* 第三层缓存（singletonFactories）: 单例工厂的缓存
```
  /** Cache of singleton objects: bean name --> bean instance */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
 
/** Cache of early singleton objects: bean name --> bean instance */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** Cache of singleton factories: bean name --> ObjectFactory */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);

```
#### `getSingleton()`
  1. Spring首先从一级缓存singletonObjects中获取。
  2. 若是获取不到，而且对象正在建立中，就再从二级缓存earlySingletonObjects中获取。
  3. 若是仍是获取不到且容许singletonFactories经过getObject()获取，就从三级缓存singletonFactory.getObject()(三级缓存)获取，若是获取到了则从三级缓存移动到了二级缓存。
  

A对象setter依赖B对象，B对象setter依赖A对象
1. A首先完成了初始化的第一步，而且将本身提早曝光到singletonFactories中
2. 发现本身依赖对象B，此时就尝试去get(B)，发现B尚未被create，因此走create流程
3. B在初始化第一步的时候发现本身依赖了对象A，因而尝试get(A)
   * 尝试一级缓存singletonObjects(确定没有，由于A还没初始化彻底)
   * 尝试二级缓存earlySingletonObjects（也没有）
   * 尝试三级缓存singletonFactories，因为A经过ObjectFactory将本身提早曝光了，因此B可以经过ObjectFactory.getObject拿到A对象(虽然A尚未初始化彻底，可是总比没有好呀)，
  

#### Spring为何不能解决非单例属性之外的循环依赖？
  
**Spring为什么不能解决构造器的循环依赖？**
* Spring解决循环依赖主要是依赖三级缓存，但是的在调用构造方法之前还未将其放入三级缓存之中
* 这类循环依赖问题可以通过使用@Lazy注解解决。
  
**Spring为什么不能解决prototype作用域循环依赖？**
* 因为spring不会缓存‘prototype’作用域的bean，而spring中循环依赖的解决正是通过缓存来实现的
  
**Spring为什么不能解决多例的循环依赖？**
* 多实例Bean是每次调用一次getBean都会执行一次构造方法并且未属性赋值，根本没有三级缓存，因此解决循环依赖。
* 这类循环依赖问题可以通过把bean改成单例的解决。
  
 
* 生成代理对象产生的循环依赖
  * 使用@Lazy注解，延迟加载
  * 使用@DependsOn注解，指定加载先后关系
  * 修改文件名称，改变循环依赖类的加载顺序
  
    
### Spring中Bean的生命周期
> Spring 只帮我们管理单例模式 Bean 的完整生命周期，对于 prototype 的 bean ，Spring 在创建好交给使用者之后则不会再管理后续的生命周期
  
<img width="644" alt="Screen Shot 2021-12-26 at 8 00 06 PM" src="https://user-images.githubusercontent.com/27160394/147407340-8b493638-02f6-4514-993d-c386e8c1b3a0.png">

1. 如果 BeanFactoryPostProcessor 和 Bean 关联, 则调用postProcessBeanFactory方法.(即首先尝试从Bean工厂中获取Bean) 如果 InstantiationBeanPostProcessor 和 Bean 关联，则调用postProcessBeforeInitialzation方法

2. Bean 容器找到配置文件中 Spring Bean 的定义 解析类得到BeanDefinition
3. Bean 容器利用 Java Reflection API 创建一个Bean的实例

4. 利用依赖注入完成 Bean 中所有属性值的配置注入

5. 调用xxxAware接口 
  * 如果 Bean 实现了 BeanNameAware 接口，调用 setBeanName()方法，传入Bean的名字
  * 如果 Bean 实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader()方法，传入 ClassLoader对象的实例。 与上面的类似，如果实现了其他 *.Aware接口，就调用相应的方法。

６. 如果有和加载这个Bean的Spring容器相关的 BeanPostProcessor对象,Spring将调用该接口的预初始化方法`postProcessBeforeInitialzation()`对Bean进行加工操作，此处非常重要，Spring的AOP就是利用它实现的

7. 如果在配置文件中通过 init-method 属性指定了初始化方法，则调用该初始化方法

8、如果有和加载这个Bean的Spring容器相关的`BeanPostProcessor`对象，执行`postProcessAfterInitialization()`方法,此时，Bean已经可以被应用系统使用了

9、如果在 <bean> 中指定了该 Bean 的作用范围为 scope="singleton"，则将该 Bean 放入 Spring IoC 的缓存池中，将触发 Spring 对该 Bean 的生命周期管理；如果在 <bean> 中指定了该 Bean 的作用范围为 scope="prototype"，则将该 Bean 交给调用者，调用者管理该 Bean 的生命周期，Spring 不再管理该 Bean

10、使用bean

11、当要销毁 Bean 的时候，如果 Bean 实现了 DisposableBean 接口，执行 destroy() 方法。
12. 如果在配置文件中通过 destory-method 属性指定了 Bean 的销毁方法，则 Spring 将调用该方法对 Bean 进行销毁
  
        
 
### Spring 中的 bean 的作用域有哪些?
  
* singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
* prototype : 每次请求都会创建一个新的 bean 实例。
* request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
* session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
* global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话

> @Component 和 @Bean 的区别是什么？

* 作用对象不同: @Component 注解作用于类，而@Bean注解作用于方法。
* @Component通常是通过类路径扫描(`@ComponentScan`)来自动侦测以及自动装配到Spring容器中。@Bean注解通常是我们在标有该注解的方法中定义产生这个 bean,@Bean告诉了Spring这是某个类的示例，当我需要用它的时候还给我。
* @Bean注解比Component 注解的自定义性更强，而且很多地方我们只能通过 @Bean 注解来注册bean。比如当我们引用第三方库中的类需要装配到 Spring容器时，则只能通过 @Bean来实现
  
> 将一个类声明为Spring的 bean 的注解有哪些?
  
我们一般使用 @Autowired 注解自动装配 bean，要想把类标识成可用于 @Autowired 注解自动装配的 bean 的类,采用以下注解可实现：
* @Component ：通用的注解，可标注任意类为 Spring 组件。如果一个Bean不知道属于哪个层，可以使用@Component 注解标注。
* @Repository : 对应持久层即 Dao 层，主要用于数据库相关操作。
* @Service : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
* @Controller : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面
