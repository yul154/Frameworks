# Spring
* Provide enterpirse development without EJBs.
* Every thing in Spring is simple POJO class
* Spring context sort of a Hashmap
# Configuration
## Main class
```
ApplicationContext appContext=new ClassPathXmlApplicationContext("ApplicationContext");// read the configuration from XML file or configuraton class
Service service=appContext.getBean("Service",Service.class);// create and return an objecy with a bean
service.findall().get(0).getFirstname();// call a bean method
```
## 1.XML
* Configuration file manage all java objects classes
* Spring context sort of a HashMap
### The XML configuration is composed of beans
* beans are basically classes
* Definding beans can be thought of as replacing `new`
* Define class, use interface
```
<bean name="customerRepository" class="com.sample.repository.HibernateCustomerRepositoryImpl" />
```
#### setup in .xml
* the ComponentScan annotation to specify the packages that we want to be scanned
```
<context:annotation-config>
<context: component-scan base-package="give package">
```
## 2. java configuration

```
@Configuration
@ComponentScan(value={"com.journaldev.spring.di.consumer"})
public class Ch2BeanConfiguration {

    @Bean
    public AccountService accountService() {
        AccountServiceImpl bean = new AccountServiceImpl();
        bean.setAccountDao(accountDao());
        return bean;
    }

    @Bean
    public AccountDao accountDao() {
        AccountDaoInMemoryImpl bean = new AccountDaoInMemoryImpl();
        return bean;
    }
}
```
* @Configuration annotation is used to let Spring know that it’s a Configuration class.
* @ComponentScan annotation is used with @Configuration annotation to specify the packages to look for Component classes.
## Bean definition
### 1. configuration inside
* based on configuration type. 
1. xml-config it will be <bean/> tag, 
2. java-based config - method with @Bean annotation and beans {...} construction for Groovy.
```
<bean name="customerRepository" class="com.sample.repository.HibernateCustomerRepositoryImpl" />
```
### 2. Annotations
### `Autowire`
* Automatically wires beans together.
* it can autowired property in a particular bean.
* find the correct match for this specific type and autowire it in.

Type|Description
----|------------
`byType`|The byType mode injects the object dependency according to type. So property name and bean name can be different. It internally calls setter method.
`byName`|The byName mode injects the object dependency according to name of the bean. In such case, property name and bean name must be same. It internally calls setter method.
`constructor`|The constructor mode injects the dependency by calling the constructor of the class. It calls the constructor having large number of parameters.
`no`| not autowired at all

### Annotation configuration
* place annotation at field, Spring will run application with stereoptype annotations and autowired annotations 
1. member injection in Service layer
2. Setter injection
3. Constructor injection
#### Stereotype Bean Annotations
* `@Componennt`:marks a java class(pojo) as a bean,tell Spring Framework - Hey there, this is a bean that you need to manage. 
* `@Controller`:marks a java class as Spring Web MVC controller
* `@Service`: Indicates that an annotated class as service layer(business logic layer)
* `@Repository`:Indicates that an annotated class as DAO laye
* @Controller, @Service, @Repository are specializations of the `Component` annotation
##### Annotation
* `@Configuration` at class leveL
*  Spring Beans defined by `@Bean`(method level)
* `@Qualifier` used to resolve the autowiring conflict, when there are multiple beans of same type. 
* `@DependsOn` can force the IoC container to initialize one or more beans
------------------------------------
# An inversion of control container
* Spring don't create an object inside another Java class(dependency), instead, rely on IOP module to create object.
* Aspect Oriented programming: Separate non-business logic and acutal businesss logic in classes
* develop loosely coupled applications
## Dependency injection
> never create an object inside another class using `new`
* Using interface to bind two classes 
* Using setter method or constructor to initialize another class object


----------------------
## Bean Scopes
* `@Scope()`
### Valid in any confiugration
#### 1. Singleton
* Default spring scope
* Single instance per Spring contanier or context
#### 2. Prototype
* unique instance per request,each time you request a bean from the container, you are guaranteed a unique instaned
### Valid only in web-aware spring project
1. Request: return a bean for each http request
2. Session: return a bean for each http session
3. Global:  return a single bean per application

------------------------------------
## Properties
* Abstract out values that can change with each environment
### 1. load a Properties file
* XML
```
<context:property-placeholder location="classpath:foo.properties" />
```
* Annotation
```
@PropertySource("classpath:foo.properties")
@PropertySource("classpath:bar.properties")
Public class name{

}
```
### 2. Extract information fomr Propertoes file
* XML
```
<bean name="customerRepository"
      class="com.sample.repository.HibernateCustomerRepositoryImpl">
   <property name="dbUsername" value-"${dbUsername}" />
</ bean>
      
```
* Annotation
```
@Value("${dbUsername}")
private String dbUsername;
```
# Spring Framework
* Core
* web
* AOP
* Data Access
* Integration
* Testing
