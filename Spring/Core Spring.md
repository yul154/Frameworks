#   What is Core Spring? 

# Core Technologies
1. Container

## The Ioc Container

Dependency Injection (DI)
* It is a process whereby objects define their dependencies only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method.
* The container then injects those dependencies when it creates the bean

`BeanFactory`
* provides an advanced configuration mechanism capable of managing any type of object

`ApplicationContext` is sub-interfae o `BeannFactory` and its adds
*  Easier integration with Spring’s AOP features
* Message resource handling (for use in internationalization)
* Event publication
* Application-layer specific contexts such as the WebApplicationContext for use in web applications.

Bean
*  the objects that form the backbone of your application and that are managed by the Spring IoC container 
* A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container. 

### Spring IoC container 

**`ApplicationContext`** interface represents the Spring IoC container 
* responsible for instantiating, configuring, and assembling the beans.
* The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata
* The configuration metadata is represented in XML, Java annotations, or Java code.

<img width="324" alt="Screen Shot 2020-08-06 at 11 24 48 AM" src="https://user-images.githubusercontent.com/27160394/89556742-750dca80-d7d7-11ea-8c2b-e6d6b4575eb1.png">


**Configuration Metadata**
* This configuration metadata represents how tell the Spring container to instantiate, configure, and assemble the objects in your application.
* Spring configuration consists of at least one and typically more than one bean definition that the container must manage.
* XML-based configuration metadata configures these beans as <bean/> elements inside a top-level <beans/>
* Java configuration typically uses `@Bean`-annotated methods within a `@Configuration` class.

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions go here -->

</beans>
```
* `id` attribute is a string that identifies the individual bean definition.
* `class` attribute defines the type of the bean and uses the fully qualified
classname.
* `property` : This attribute is used to inject the dependencies through setter method.
* `ref` element refers to the name of another bean definition
* `<import/>` element to load bean definitions from another file or files

**1. Instantiating a Container**
```
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```
* The paths supplied to an ApplicationContext constructor are resource strings that let the container load configuration metadata from a variety of external resources,


**2. Using the container**
* `getBean(String name, Class<T> requiredType)`: retrieve instances of your beans.
```
PetStoreService service = context.getBean("petStore", PetStoreService.class);
List<String> userList = service.getUsernameList();
```


### Bean

1. **Naming Bean**
* Every bean has one or more identifier,These identifiers must be unique within the container that hosts the bean
* Use the `id` attribute, the `name` attribute, or both to specify the bean identifiers.
* If you want to introduce other aliases for the bean, you can also specify them in the `name` attribute

2. **Instantiating Beans**
* If you use XML-based configuration metadata, you specify the type (or class) of object that is to be instantiated in the class attribute of the <bean/> element.
 1. Typically, to specify the bean class to be constructed in the case where the container itself directly creates the bean by calling its constructor reflectively, somewhat equivalent to Java code with the new operator.
 2. To specify the actual class containing the static factory method that is invoked to create the object, in the less common case where the container invokes a static factory method on a class to create the bean. 
* Instantiation with a Constructor
  1. When you create a bean by the constructor approach, all normal classes are usable by and compatible with Spring
  2. Simply specifying the bean class should suffice
* Instantiation with a Static Factory Method
  1. use the `class` attribute to specify the class that contains the static factory method
  2. `factory-method` to specify the name of the factory method itself.
```
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
    
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```
* Instantiation by Using an Instance Factory Method
 1. leave the `class` attribute empty and,
 2. in the `factory-bean` attribute, specify the name of a bean in the current (or parent or ancestor) container that contains the instance method that is to be invoked to create the object. 
 3. Set the name of the `factory method` itself with the factory-method attribute.
```
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

### Dependencies

#### Dependency Injection
> a process whereby objects define their dependencies
>> only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. 

Benefits
* Code is cleaner with the DI principle
* Decoupling is more effective when objects are provided with their dependencies. 
* The object does not look up its dependencies
* Does not know the location or class of the dependencies. 
* your classes become easier to test

**1.Constructor-based Dependency Injection**
Constructor-based DI is accomplished by the container invoking a constructor with a number of arguments, each representing a dependency.
