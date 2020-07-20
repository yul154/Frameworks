# What is Spring boot?
* Spring Boot is another Java framework from Sring umbrella which aims to simplify the use of Spring Framework for Java development. It removes most of the pain associated with dealing with Spring e.g. a lot of configuration and dependencies and a lot of manual setups.


# Advantage of Spring Boot 
 
Everything is auto configured; no manual configurations are needed.
* auto-configuration is disabled by default and you need to use either @SpringBootApplication or @EnableAutoConfiguration annotations on the Main class to enable the auto-configuration feature. 
* Spring boot auto configuration scans the classpath, finds the libraries in the classpath and then attempt to guess the best configuration for them, and finally configure all such beans.
*  Spring Boot auto-configuration automatically configure a Spring application based on the dependencies present on the classpath. Spring Boot detects classes in the classpath and auto-configuration mechanism will ensure to create and wires necessary beans for us


Eases dependency management
* Starter dependencies: you can just specify spring-boot-web-starter and it will automatically download both those JAR files
* Thereis is no No version conflict. you do not need to provide version information into child dependencies.
* no longer need to worry about the dependencies and they will be automatically managed by Spring Boot Starters.      
```
<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.9.RELEASE</version>
	</parent>
 
 <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```

Embedded server
* Spring boot applications always include tomcat as embedded server dependency. It means you can run the Spring boot applications from the command prompt without needling complex server infrastructure.
*  you don't need to setup a Tomcat server to run your web application. You can just write code and run it as Java application 

Provide a more way to initial setup your application by Using Spring Initializer
* a web application which helps you to create initial Spring boot project structure and provides Maven or Gradle build file to build your code.
* It helps you to customize your projects with configurations and manage Spring Boot dependencies.

# what is .property file in Spring boot
> environment-specific configuration 
* simple key-value storage for configuration properties
* let developer externalize configuration so that developer can work with the same application code in different environments

Common application properties
* Server.port
* spring.application.name: registering with a service registry
* spring.profiles.active
* spring.autoconfigure.exclude
* logging.config
* server.address 

# If you have to make spring boot from scratch how will take it 
1. Set up project by Spring initializr
  * Basic configure our spring boot application

2. import project which made by initialize

3. set up pom.xml(add dependency)
* Use starters to conveniently management dependencies.

4. configure application in application.properties File(Setting database Connection Properties)

5. Write the Application
   * Add @SpringBootApplication annotation in the main class
   
5. run The main() method uses Spring Boot’s SpringApplication.run() method to launch an application
   
# Spring Boot annotation 

@SpringBootApplication :Indicates a configuration class that declares one or more @Bean methods and also triggers auto-configuration and component scanning.
* Combination of @Configuratio,@EnableAutoConfiguration,@ComponentScan

@Configuration: Tags the class as a source of bean definitions for the application context.

@EnableAutoConfiguration: Tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings. For example, if spring-webmvc is on the classpath, this annotation flags the application as a web application and activates key behaviors, such as setting up a DispatcherServlet.

@ComponentScan: Tells Spring to look for other components, configurations, and services in the com/example package, letting it find the controllers.

@Conditional:  can be used for conditional bean registrations.

@profile :used to write conditional checking based on Environment variables.Profiles can be used for loading application configuration based on environments.



