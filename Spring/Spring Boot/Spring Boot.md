# Spring Boot  Overview
* It is used to solve same problems like Spring Framework.
* devtools: automatically restart whenever files on the classpath change
* It provides Embedded HTTP server
## WorkFlow
1. `public static void main()`--> starts java and then the application
2. `@SpringBootApplication`:Spring configuration on startup, Auto Configures framework, Scans projects for Spring components
3. 
## Feature
### 1. Automatic configuration
> Automatically configure the application based on libraries added to project
* Absolutely no code generation and no requirement for XML configuration
* look at the jars on the classpath
* Auto-configures beans
* Automatic configuration enable, diabled or excluded or always on regardless of any condition
#### 1.1 Annotation
> add to the main application class to indicate application should utilize Spring Boot
##### 1.1.1 `@SpringBootConfiguration` in main class
* replaces `@Configuration` to annotates a class as configurastion
* replaces `@EnableAutoConfiguration`: tells Spring Boot to configure beans
* replaces `@ComponentScan`: tells Spring Boot to scan current package and subpackages
* To create a deployable way, we should extend our main application class by `SpringBootServletInitializer`.
##### 1.1.2 `Conditionalon`
* `ConditonalOnClass`:only apply this configuration if this class is present on the classpath
* `@ConditionalOnMissingBean`:registering beans only when they are not already in the application context.
* `@ConditionalOnBean`:only apply this configuration if the bean defined within the application context.
* `@ConditionalOnProperty`:conditionally create a Spring bean depending on the configuration of a property.
#### 1.1.3 Custom auto-configuration
*  need to create a class annotated as @Configuration and register it.
*  registering the class as an auto-configuration candidate
#### 1.2 Dependency management
* `spring-boot-starter-parent`provides a dependency-management section so that you can omit version tags
* spring initializr: generates spring boot project with just what you need to start quickly

#### 1.3 Spring Boot externalized Configuration
* externalize your configuration so that you can work with the same application code in different environments
##### 1.3.1 Spring Boot Property
* `application.properties` file is offical Spring boot configuration file
* add specific configuration for applications
* simple key-value storage for configuration properties
* Common application properties. 
[https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html]
##### 1.3.2 YAML
* Support Key/Value(map),Lists and Scalar types
  * yml uses ` -value` or `numbers[one,two]` for a list
* Used in many languages
* Hierarchical structure，use two space one line to specify key-value pairs or(inline map)
* Doesn' work with @PropertySource
* Mulriple Spring Profiles in one singl default config
* yml keys are Strings and values are their respective type
##### 1.3.3 Typesafe Configuration
> letting developer maps the entire .properties and yml file into an object easily.
* `@ConfigurationProperties`: 
  * define all of application configuartion as a simple typesafe POJO
  * Add this to a class definition or a @Bean method in a @Configuration class if you want to bind and validate some external Properties
##### 1.3.4 relaxed binding
* there does not need to be an exact match between the Environment property name and the bean property name
* Camel case: 

#### 1.4 Spring Boot profile
* provide a way to segregate parts of your application configuration and make it be available only in certain environments
* `@Profile`:map our beans to different profiles
* Any @Component or @Configuration can be marked with @Profile to limit when it is loaded
* .properties keys and values are Strings, uses the array syntax as a convention t specify a list
```
numbers[0]=one
numbers[1]=two
numbers=one,two//inline
```
* .properites uses dots t denote hierarchy
* 
-------------
### 2. Starter dependencies
> simplify your Maven configuration
* Provide opinionated 'starter' POMs to simplify your Maven configuration
* Integrate multiple dependencies into one 
--------------------
### 3.Command line interface(CLI)
> rapidly prototype your applications
* application written using Groovy scripts
* Create Groovy file that contains details about the required application
* use `curl`
-----------------------------
### 4.Actuator
> Manage and Monitor application
* allow you to see what's going on inside of your running spring boot application
* by using http endpoints or JMX
* see application health status, metrics, configured logger
#### 4.1 Endpoints
* autoconfig reports
* beans: Displays a complete list of all the Spring beans in your application.
* configprops:Displays a collated list of all @ConfigurationProperties.
#### 4.1.1 health:Shows application health information.
* RedisHealthIndicator: check that you have a connection to the Redis
* DiskSpaceHealthIndicator: checks available disk space and reports a status of Status.DOWN when it drops below a configurable threshold.
* DataSourceHealthIndicator: tests the status of a DataSource and connectivity to database
* ElasticsearchHealthIndicator: application can get access to the Elasticsearch server
* CustomHealthIndicator
```
public class MyHealthIndicator implements HealthIndicator {

	@Inject Properties properties;
  
  @Override
	public Health health() {
		int errorCode = check(); // perform some specific health check
		if (errorCode != 0) {
			return Health.down().withDetail("Error Code", errorCode).build();
		}
		return Health.up().build();
	}

}
```
---------------------
#  Access Data With Spring Boot and H2
## H2
* databases writte in Java
* In-memory database
## flyway
> Database-migration tools
## ORM with JPA
1. persistence data store
2. JDBC: Java API to connect and execute queries against the database
3. JPA: on top of JDBC, makes it easy to map java objects to relational databases 
4. Spring Data JPA: provides repository support for the Java Persistence API (JPA)
* `spring-boot-starter-data-jpa`: provide Hibernate,Spring data JPA, Spring ORM dependencies 
## Entity
> Objects live in a database
* mapped to a database
### Annotation
* `@Entity` : indicating that it is a JPA entity class
* `@Id`:JPA Primary Key,JPA will recognize it as the object’s ID

-------------------------------
# Configurate Spring MVC
* dependency `spring-boot-starter-web`: Dispatcher Sevlet, error page, servlet container, jars
### 1. Model
* represent entity or domain in system
* model is data stored in a database
* represented as a java object
* `@Entity` : indicating that it is a JPA entity class
### 2. View
* Typically done in JSP,JSF or ThymeLeaf
* Spring does that via the view resolvers, which enable you to render models in the browser without tying the implementation to a specific view technology.
* register view resolvers is unnecessary in Spring boot, Add entries in application.properties 
### 3. Controller
* Directing incoming user requests to correct resources
* Sending responses from those resources back to user
#### 3.1 Actions in Spring boot 
```
@RequestMapping(value = "/get/{id}", method = RequestMethod.GET)
```
```
@GetMapping("/get/{id}")
```
--------------------
# Build RESTful application with Spring Boot
> REST is a popular way to expose data from a server through an API 
* Using `@RestController` annotate the controller now simply returns object data that is written directly to the HTTP request as a given form
* `@RestController`== `@Controller`+`@ResponseBody`+RESTful service
* `@ResponseBody`:return value of the method is serialized directly to the body of the HTTP requests.
* ResponseEntity: represent the entire HTTP response
* `ResponseStatusException`: Programmatic alternative to @ResponseStatus
## `spring-boot-starter-web` : 
* automatically sets up ViewResolvers
* Sets up static resource serving
* Set up HttpMessageConverter
## Properties
> Set up environment externally
* externalize configuration so that can work with the same application code in different environments
* Property values can be injected directly into your beans by using the @Value
* Spring boot's going to load all of main application properties first, and then override any environment-specific properties as needed
* ability to define additio property files that have a profile embedded name of the file
------------------
# Build a GraphQL Server
> Query language for APIs that describes how to ask for data
* GraphQL offers greater flexibility in the response returned than REST
* Allows client to specify the exact data needed
* Mutations 
```
@Component
public class Query implements GraphQLQueryResovler
```
----------------
# Testing in Spring Boot
* `WebMvcTest`: used for controllerlayer unit testing,dependent beans must be mocked
* mockMvc: quickly test MVC controllers without needing to start a full HTTP server
* `SpringBootTest`: used for integration testing

# App on Cloud
## Docker
> Virtualization management software for containers and images
* Build images which is software that run within the container
* Deploy images into containers which is run environment
* Manage containers
