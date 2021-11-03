# Catalog 
1. Spring’s bean container
2. Exploring Spring’s core modules 
3. The greater Spring ecosystem
4. What’s new in Spring
---
## 1.1 Simplifying Java development

*  Lightweight and minimally invasive development with POJOs
*  Loose coupling through DI and interface orientation
*  Declarative programming through aspects and common conventions 
*  Eliminating boilerplate code with aspects and templates

### 1.1.1 Unleashing the power of POJOs

Spring’s non-invasive programming model means this class could function equally well in a Spring application as it could in a non-Spring application.


### 1.1.2 Injecting dependencies

#### HOW DI WORKS
Traditionally, each object is responsible for obtaining its own references to the objects it collaborates with (its dependencies).(highly coupled)

With DI, objects are given their dependencies at creation time by some third party that coordinates each object in the system.
  * Objects aren’t expected to create or obtain their dependencies

 Doesn’t `new` his own referennce.
 * Instead, given a reference at construction time as a constructor argument. This is a type of DI known as constructor injection.
 *  It isn’t coupled to any specific implementation by injecting an interface.
 
 If an object only knows about its dependencies by their interface (not by their implementation or how they’re instantiated), then the dependency can be swapped out with a different implementation without the depending object knowing the difference.
 
 #### INJECTING
 *wiring* : The act of creating associations between application components is commonly referred to 
 
 #### LOADING
> an *application context* loads bean definitions and wires them together. 

### 1.1.3 Applying aspects

aspect oriented programming (AOP) enables you to capture functionality that’s used throughout your application in reusable components.

often These components also carry additional responsibilities beyond their core functionality. 

cross cutting concerns: System services such as logging, transaction management, and security often find their way into components whose core responsibilities is something else.

Cross cutting concerns two levels of complexity 
* system-wide concerns code is duplicated across multiple component,
  *  change all places if need
  *  Abstracted the concern to a separate module so that the impact to your components is a single method call, that method call is duplicated in multiple places.

* Your components are littered with code that isn’t aligned with their core func- tionality. 

AOP makes it possible to modularize these services and then apply them declaratively to the components they should affect.

### 1.1.4 Eliminating boilerplate code with templates
> Spring seeks to eliminate boilerplate code by encapsulating it in template
---
## 1.2 Containing your beans
> In a Spring-based application, your appli- cation objects live in the Spring containe

**the container(IOC)**
* creates the objects
* wires them together
* configures them
* manages their complete lifecycle from cradle to grave

Spring comes with several container implementations that can be categorized into two distinct types
* *Bean factories* (defined by the `org.springframework.beans.factory.BeanFactory` interface) are the simplest of containers, providing basic support for DI. 
* *Application contexts*  build on the notion of a bean factory by providing applicationframework services,

### 1.2.1 Working with an application context

### 1.2.2 A bean's life

![Screen Shot 2021-11-01 at 11 35 55 PM](https://user-images.githubusercontent.com/27160394/139698220-48493561-587f-4530-a6c4-c17f40a6c98e.png)

![Screen Shot 2021-11-01 at 11 36 34 PM](https://user-images.githubusercontent.com/27160394/139698307-4fcc020d-e90e-4695-befd-f84550d94d3f.png)

1. Spring instantiates the bean.
2. Spring injects values and bean references into the bean’s properties.
3. If the bean implements BeanNameAware, Spring passes the bean’s ID to the setBeanName() method.
4. If the bean implements BeanFactoryAware, Spring calls the setBeanFactory() method, passing in the bean factory itself.
5. If the bean implements ApplicationContextAware, Spring calls the setApplicationContext() method, passing in a reference to the enclosing appli-
cation context.
6. If the bean implements the BeanPostProcessor interface, Spring calls its postProcessBeforeInitialization() method.
7. If the bean implements the InitializingBean interface, Spring calls its afterPropertiesSet() method. Similarly, if the bean was declared with an init-
method, then the specified initialization method is called.
8. If the bean implements BeanPostProcessor, Spring calls its postProcessAfterInitialization() method.
9. At this point, the bean is ready to be used by the application and remains in the application context until the application context is destroyed.
10. If the bean implements the DisposableBean interface, Spring calls its destroy() method. Likewise, if the bean was declared with a destroy-method,
the specified method is called.

---
# Wiring beans
* Declaring beans
* Injecting constructors and setters
* Wiring beans
* Controlling bean creation and destruction

In Spring, objects aren’t responsible for finding or creating the other objects that they need to do their jobs. Instead, the container gives them references to the objects that they collaborate with.

## 2.1 Exploring Spring’s configuration options
as a developer to tell Spring which beans to create and how to wire them together

* Explicit configuration in XML
*  Explicit configuration in Java
* Implicit bean discovery and automatic wiring
---
## 2.2 Automatically wiring beans

Spring attacks automatic wiring from two angles:
* *Component scanning* — Spring automatically discovers beans to be created in the application context.
* *Autowiring* - Spring automatically satisfies bean dependencies.

### 2.2.1 Creating discoverable beans
* it keeps the coupling between any CD player implementation and the CD itself to a minimum
```
package soundsystem;
public interface CompactDisc { // an interface that defines a CD.
  void play();
}

```
* implementation of `CompactDisc`

```
package soundsystem;
import org.springframework.stereotype.Component;
@Component
public class SgtPeppers implements CompactDisc {
   private String title = "Sgt. Pepper's Lonely Hearts Club Band"; 
   private String artist = "The Beatles";
   public void play() {
        System.out.println("Playing " + title + " by " + artist);
   } 
}
```
* `@Component`: identifies this class as a component class and serves as a clue to Spring that a bean should be created for the class
*  `@ComponentScan` : will default to scanning the same package as the configuration class.

### 2.2.2 Naming a component-scanned bean
Spring given a name for bean from its class name by lowercasing the first letter of the class name.

* If you’d rather give the bean a different ID,
```
@Component("lonelyHeartsClub")
public class SgtPeppers implements CompactDisc
```
* Use the `@Named` annotation from the Java Dependency Injection specification  to provide a bean ID
```
@Named("lonelyHeartsClub")
public class SgtPeppers implements CompactDisc
```
### 2.2.3 Setting a base package for component scanning
`@ComponentScan` will default to the configuration class’s package as its base package to scan for components

One common reason for explicitly setting the base package is so that you can keep all of your configuration code in a packag
```
@ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class})
```
### 2.2.4 Annotating beans to be automatically wired
 Autowiring is a means of letting Spring automatically satisfy a bean’s dependencies by finding other beans in the application context that are a match to the bean’s needs.
 
`@Autowired`indicate that autowiring should be performed
 * annotation’s use isn’t limited to constructors.
 * It can also be used on a property’s setter method
 * Spring will attempt to satisfy the dependency expressed in the method’s parameters.
```
@Autowired//Spring creates the CDPlayer bean, it should instantiate it via that constructor
public CDPlayer(CompactDisc cd){} // pass in a bean that is assignable to CompactDisc.
```
```
@Autowired
public void setCompactDisc(CompactDisc cd)
```
* If there are no matching beans,, Spring will throw an exception. To avoid that exception, you can set the required attribute on @Autowired to false
```
@Autowired(required=false)
public CDPlayer(CompactDisc cd)
```
Spring supports the `@Inject` annotation for autowiring alongside its own `@Autowired`
---
## 2.3 Wiring beans with Java
JavaConfig is the preferred option for explicit configura- tion because it’s more powerful, type-safe, and refactor-friendly

### 2.3.1 Creating a configuration class
With `@ComponentScan` gone
* the Config class is ineffective. If you were to run Test now, the test would fail with a `BeanCreationException`

### 2.3.2 Declaring a simple bean
> creates an instance of the desired type and annotate it with `@Bean`
```
@Bean
public CompactDisc sgtPeppers() {
  return new SgtPeppers();
}
```
you can either rename the method or prescribe a different name with the name attribut
```
@Bean(name="lonelyHeartsClubBand")
public CompactDisc sgtPeppers()
```
### 2.3.3 Injecting with JavaConfig
The simplest way to wire up beans in JavaConfig is to refer to the referenced bean’s method. 
```
@Bean
public CDPlayer cdPlayer() {
  return new CDPlayer(sgtPeppers());
}
```
* Spring will intercept any calls to it and ensure that the bean produced by that method is returned rather than allowing it to be invoked again.

Bean as a parameter
```
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc
```
-----
# 3.Advanced wiring
