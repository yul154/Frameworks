## This chapter covers
* Fine-tuning autoconfigured beans
* Applying configuration properties to application components
* Working with Spring profiles
---
## 5.1 Fine-tuning autoconfiguration
There are two different (but related) kinds of configurations in Spring:
* Bean wiring : Configuration that declares application components to be created as beans in the Spring application context and how they should be injected into each other.
* Property injection : Configuration that sets values on beans in the Spring application context.

### 5.1.1 Understanding Spring’s environment abstraction
The Spring environment pulls from several property sources,
* JVM system properties
* Operating system environment variables 
* Command-line arguments
* Application property configuration files

The Spring environment pulls properties from property sources and makes them available to beans in the application context.
<img width="363" alt="Screen Shot 2021-10-14 at 1 17 05 PM" src="https://user-images.githubusercontent.com/27160394/137256195-c4ac9974-ed1e-4563-af83-c11f6e5313e1.png">

### 5.1.2 Configuring a data source
```
datasource:
    jdbcUrl: ${spring.datasource.url}
    password: Eason@123
    url: jdbc:mysql://localhost:3306/device_config
    username: root
```
*  specify the data- base initialization scripts
```
spring:
  datasource:
    schema:
    - order-schema.sql
    - ingredient-schema.sql 
    - taco-schema.sql
    - user-schema.sql
  data:
    - ingredients.sql
```
### 5.1.3 Configuring the embedded server

 Set it up to handle HTTPS requests.
 1. create a keystore using the JDK’s keytool
 2. set a few properties to enable HTTPS in the embedded server.
```
server:
       port: 8443
       ssl:
          key-store: file:///path/to/mykeys.jks  //the path where the keystore file is created
          key-store-password: letmein 
          key-password: letmein
```
### 5.1.4 Configuring logging
```
logging:
  path: /var/logs/ 
  file: TacoCloud.log
  level:
    root: WARN
    org:
      springframework:
        security: DEBUG
```
### 5.1.5 Using special property values
Use the `${}` placeholder markers to extract property  values
---
## 5.2 Creating your own configuration properties
`@ConfigurationProperties`:support property injection of configuration properties

### 5.2.1 Defining configuration properties holders
`@ConfigurationProperties` are in fact often placed on beans whose sole purpose in the application is to be holders of configuration data.
```

@Component 
@ConfigurationProperties(prefix="taco.orders") @Data
@Validated
public class OrderProps {
  @Min(value=5, message="must be between 5 and 25")
  @Max(value=25, message="must be between 5 and 25")
  private int pageSize = 20;
}
```
### 5.2.2.Declaring configuration property metadata
There’s missing metadata concerning the configuration propert
* To create metadata for your custom configuration properties, you’ll need to create a file under the META-INF
* named `additional-spring-configuration-metadata.json`. 
---
## 5.3 Configuring with profiles
> One way to configure properties uniquely in one environment over another is to use environment variables to specify configuration properties 
Profiles are a type of conditional configuration 
* where different beans, configuration classes, and configuration properties are applied 
* or ignored based on what profiles are active at runtime.

### 5.3.1 Defining profile-specific properties
One way to define profile-specific properties is to create yet another YAML or properties file containing only the properties for different environment.
* name: application-{profile name}

Another way to specify profile-specific properties  works only with YAML configuration.
* separated by three hyphens and the `spring.profiles` property to name the profile.
```
---
spring:
  profiles: dev
```
### 5.3.2 Activating profiles
```
 profiles:
    active: dev
```

### 5.3.3 Conditionally creating beans with profiles
There are some beans that you only need to be created if a certain profile is active
  * the `@Profile` annotation can designate beans as only being applicable to a given profile.
  * Annotate the `CommandLineRunner` bean method with `@Profile` like this:
 ```
 
@Bean
// @Profile("dev")
@Profile({"dev", "qa"})
public CommandLineRunner dataLoader(){};
}
 ```




