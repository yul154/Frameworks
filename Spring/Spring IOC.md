# Spring IOC

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


## IOC的底层实现
Spring IoC 的底层实现是基于反射技术
* 工厂+反射+配置文件(bean)
* 使用反射技术获取对象的信息包括：类信息、成员、方法等等
* 再通过 xml 配置 或者 注解 的方式，说明依赖关系
* 在调用类需要使用其他类的时候，不再通过调用类自己实现，而是通过 IoC 容器进行注入。

### Spring IOC容器是怎么实现对象的创建和依赖的
1. 根据Bean配置信息在容器内部创建Bean定义注册表
2. 根据注册表加载、实例化bean、建立Bean与Bean之间的依赖关系
3. 将这些准备就绪的Bean放到Map缓存池中，等待应用程序调用

Spring容器(Bean工厂)可简单分成两种：
* BeanFactory :这是最基础、面向Spring的
* ApplicationContext : ApplicationContext是BeanFactory的子类



### Ioc 配置的三种方式


**1.xml 配置**

将bean的信息配置.xml文件里，通过Spring加载文件为我们创建bean

**2.Java 配置**

将类的创建交给我们配置的JavcConfig类来完成，Spring只负责维护和管理，采用纯Java创建方式。其本质上就是把在XML上的配置声明转移到Java配置类中


**3.注解配置**

通过在类上加注解的方式，来声明一个类交给Spring管理，Spring会自动扫描带有@Component，@Controller，@Service，@Repository这四个注解的类，然后帮我们创建并管理，前提是需要先配置Spring的注解扫描器

1. 对类添加@Component相关的注解，比如@Controller，@Service，@Repository 
2. 设置ComponentScan的basePackage,`@ComponentScan("tech.pdai.springframework")`注解，或者 `new AnnotationConfigApplicationContext("tech.pdai.springframework"`)指定扫描的basePackage.


## DI
> 控制反转是通过依赖注入实现的，其实它们是同一个概念的不同角度描述。通俗来说就是IoC是设计思想，DI是实现方式


DI的关键是：“谁依赖谁，为什么需要依赖，谁注入谁，注入了什么
* 被注入的对象”依赖“依赖对象
* 容器管理对象需要IoC容器来提供对象需要的外部资源
* 很明显是IoC容器注入某个对象，也就是注入“依赖对象”
* 就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）


###   依赖注入方式
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



#### 常用的自动装配注解有以下几种

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




> Spring什么时候实例化bean

1. 如果你使用`BeanFactory`作为Spring Bean的工厂类，则所有的bean都是在第一次使用该Bean的时候实例化
2.如果你使用ApplicationContext作为Spring Bean的工厂类，则又分为以下几种情况：
   * 如果bean的scope是singleton的，并且lazy-init为false（默认是false，所以可以不用设置），则ApplicationContext启动的时候就实例化该Bean，并且将实例化的Bean放在一个map结构的缓存中，下次再使用该Bean的时候，直接从这个缓存中取 
   * 如果bean的scope是singleton的，并且lazy-init为true，则该Bean的实例化是在第一次使用该Bean的时候进行实例化 
   * 如果bean的scope是prototype的，则该Bean的实例化是在第一次使用该Bean的时候进行
