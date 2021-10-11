# 1 Getting started with Spring
* Spring and Spring Boot essentials
* Initializing a Spring project
* An overview of the Spring landscape
---
##  1.1 What is Spring?
Spring offers a *containe*, often referred to as the *Spring application context*
 * Creates and manages application components. 
 * These components, or *beans*, are wired together inside the Spring application context 

*dependency injection*
* The act of wiring beans together is based on a pattern
* Rather than have components create and maintain the lifecycle of other beans that they depend on
* Relies on a separate entity (the container) to create and maintain all components and inject those into the beans that need them. 

```
@Configuration
public class ServiceConfiguration {
  @Bean
  public InventoryService inventoryService() {
    return new InventoryService();
  }
  @Bean
  public ProductService productService() {
    return new ProductService(inventoryService());
  }
}
```
* The `@Configuration` annotation indicates to Spring that this is a configuration class that will provide beans to the Spring application context
* `@Bean`, indicating that the objects they return should be added as beans in the application context


*component scanning*
* With component scanning, Spring can automatically discover components from an application’s classpath and create them as beans in the Spring application context.

*autowiring*
* Spring automatically injects the components with the other beans that they depend on.

*autoconfiguration*
* Spring Boot can make reasonable guesses of what components need to be configured and wired together, based on entries in the classpath, environment variables, and other factors.
---
## 1.2 Initializing a Spring application
### 1.2.2 Examining the Spring project structure
<img width="434" alt="Screen Shot 2021-10-11 at 3 19 13 PM" src="https://user-images.githubusercontent.com/27160394/136748484-482bb920-a7e5-45ec-8abb-1a8c7391fd5b.png">

* `mvnw` and `mvnw.cmd` -> These are Maven wrapper scripts.  Use these scripts to build your project even if you don’t have Maven installed on your machine.
* `pom.xml`:This is the Maven build specification.
*  `TacoCloudApplication.java` ->  Spring Boot main class that bootstraps the project.
*  application.properties -> This file is initially empty, where can specify configuration properties
*  *static* -> This folder is where you can place any static content (images, stylesheets, JavaScript, and so forth) that you want to serve to the browser.
*  *templates* -> Is a place where you put all the thymeleaf templates that will be used to render content to the browse
*  `TacoCloudApplicationTests.java` -> This is a simple test class that ensures that the Spring application context loads successfully.


**`pom.xml`**

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Starter dependencies offer three primary benefits:
* Your build file will be significantly smaller and easier to manage
* You’re able to think of your dependencies in terms of what capabilities they provide, rather than in terms of library names.
* You’re freed from the burden of worry about library versions.

```
<build>
 <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```
This plugin performs a few important functions:
* It provides a Maven goal that enables you to run the application using Maven. 
* It ensures that all dependency libraries are included within the executable JAR file and available on the runtime classpath.
* It produces a manifest file in the JAR file that denotes the bootstrap class (TacoCloudApplication） as the main class for the executable JAR.


`@SpringBootApplication ` combines three other annotations:
* `@SpringBootConfiguration`:Designates  this class as a configuration class.a specialized form of the `@Configuration` annotation.
* `@EnableAutoConfiguration`: tells Spring Boot to automatically configure any components that it thinks you’ll need.
* `@ComponentScan` : Spring can automatically discover class with annotations ` @Component, @Controller, @Service` and register them as components in the Spring application context.

*TESTING THE APPLICATION*

` @RunWith(SpringRunner.class)`

*  `@RunWith` is a JUnit annotation, providing a  test runner that guides JUnit in running a test
*  `SpringRunner` is an alias for `SpringJUnit4ClassRunner`
*  `@SpringBootTest` tells JUnit to bootstrap the test with Spring Boot capabilities.
---
## 1.3 Writing a Spring application
### 1.3.1 Handling web requests
*controller* -> a class that handles requests and responds with information of some sort.
  *  Responds by optionally populating model data and passing the request on to a view to produce HTML that’s returned to the browser.

```
package tacos;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller // identify this class as a component for com- ponent scanning.
public class HomeController {
	@GetMapping("/") // indicate that if an HTTP GET request is received for the root path /,
	public String home() {
		
		return "home";
	}

}
```
### 1.3.2 Defining the view
*WHY THYMELEAF?*
* The template name is derived from the logical view name by prefixing it with */templates/* and postfixing it with *.html*. 

### 1.3.3 Testing the controller
```
@RunWith(SpringRunner.class) 
@WebMvcTest(HomeController.class)
public class HomeControllerTest {
	
	@Autowired
	private MockMvc mockMvc;

	@Test
	public void testHomePage() throws Exception{
		mockMvc.perform(get("/")) //Performs GET 
		.andExpect(status().isOk())
		.andExpect(view().name("home"))
		.andExpect(content().string( containsString("Welcome to...")));
		
		
	}
}
```
`@WebMvcTest` :  arranges for the test to run in the context of a Spring MVC application.

From that request, it sets the following expectations:
* The response should have an HTTP 200 (OK) status.
* The view should have a logical name of home.
* The rendered view should contain the text “Welcome to....”

### 1.3.4 Building and running the application
<img width="314" alt="Screen Shot 2021-10-11 at 4 05 22 PM" src="https://user-images.githubusercontent.com/27160394/136754458-bf56b09c-b2ee-44b5-9db9-573b221f97b7.png">

### 1.3.5 Getting to know Spring Boot DevTools
DevTools provides Spring developers with some handy develop- ment-time tools.
#### 1.Automatic application restart when code changes
* when DevTools is in play, the application is loaded into two sepa- rate class loaders in the Java virtual machine
* One class loader is loaded anything that’s in the`src/main/` path of the project
* The other class loader is loaded with dependency libraries
*  DevTools reloads only the class loader containing your project code and restarts the Spring application context

#### 2.Automatic browser refresh when browser-destined resources (such as templates, JavaScript, stylesheets, and so on) change, Automatic disable of template caches
* DevTools addresses this issue by automatically disabling all template caching.
* it automatically enables a `LiveReload` server along with your application,it causes your browser to automatically refresh when changes are made to templates

#### 3. Built in H2 Console if the H2 database is in use
* DevTools will also automatically enable an H2 Console that you can access from your web browser. 
* You only need to point your web browser to http://localhost:8080/h2-console to gain insight into the data your application is working with.
---
## 1.4 Surveying the Spring landscape
### 1.4.1 The core Spring Framework
* It provides the core container and dependency injection framework
### 1.4.2 Spring Boot
*  The Actuator provides runtime insight into the inner workings of an application
*  Flexible specification of environment properties.
*  Additional testing support on top of the testing assistance
### 1.4.3 Spring Data
* The ability to define your application’s data repositories as simple Java interface
* Using a naming convention when defining methods to drive how data is stored and retrieved.
* Spring Data is capable of working with a several different kinds of databases

### 1.4.4 Spring Security
* Spring Security addresses a broad range of application security needs, including authentication, authorization, and API security.
### 1.4.5 Spring Integration and Spring Batch
* Spring Integration provide the implementation of these patterns for integrate with other applications or even with other components of the same application
* Spring Batch addresses batched integration where data is allowed to collect for a time until some trigger signals that it’s time for the batch of data to be processed.

### 1.4.6 Spring Cloud
* compose applications from several individual deployment units known as microservices.
* 

* 

*  







