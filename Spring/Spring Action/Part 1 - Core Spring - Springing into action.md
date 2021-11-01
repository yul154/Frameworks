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

