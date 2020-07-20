# Spring Boot Annotations
> `org.springframework.boot.autoconfigure` and `org.springframework.boot.autoconfigure.condition`

`@SpringBootApplication`: mark the main class of a Spring Boot application.

`@SpringBootApplication` = `@Configuration` + `@ComponentScan` + `@EnableAutoConfiguration`

Let's understand about each annotation

* `@EnableAutoConfiguratio`n : enables auto-configuration. It means that Spring Boot looks for auto-configuration beans on its classpath and automatically applies them.
* `@Configuration` : It indicates that the class has @Bean definition methods by defining methods with the @Bean annotation. As a result, Spring container can process the class and produce Spring Beans to be used in the application
* `@ComponentScan` : @ComponentScan without arguments indicates Spring to scan the current package and all of its sub-packages.



## Auto-Configuration Conditions
We can place the annotations in this section on @Configuration classes or @Bean methods.

`@ConditionalOnClass` and `@ConditionalOnMissingClass` : Using these conditions, Spring will only use the marked auto-configuration bean if the class in the annotation's argument is present/absent:
```
@Configuration
@ConditionalOnClass(DataSource.class)
class MySQLAutoconfiguration {
    //...
}
```

`@ConditionalOnBean` and `@ConditionalOnMissingBean` : when we want to define conditions based on the presence or absence of a specific bean:

```
@Bean
@ConditionalOnBean(name = "dataSource")
LocalContainerEntityManagerFactoryBean entityManagerFactory() {
    // ...
}
```

`@ConditionalOnProperty` : we can make conditions on the values of properties:

```
@Bean
@ConditionalOnProperty(
    name = "usemysql", 
    havingValue = "local"
)
DataSource dataSource() {
    // ...
}
```

`@ConditionalOnResource`: only when a specific resource is present:
```
@ConditionalOnResource(resources = "classpath:mysql.properties")
Properties additionalProperties() {
    // ...
}
```

`@ConditionalOnWebApplication` and `@ConditionalOnNotWebApplication`:  we can create conditions based on if the current application is or isn't a web application:

`@ConditionalExpression`: Spring will use the marked definition when the SpEL expression is evaluated to true
```
@Bean
@ConditionalOnExpression("${usemysql} && ${mysqlserver == 'local'}")
DataSource dataSource() {
    // ...
}
```

` @Conditional`:create a class evaluating the custom condition. 
```
@Conditional(HibernateCondition.class)
Properties additionalProperties() {
    //...
}
```
---

# Spring Boot Starters

Starter POMs are a set of convenient dependency descriptors that you can include in your application.get a one-stop-shop for all the Spring and related technology that you need

Spring Boot starters can help to reduce the number of manually added dependencies,So instead of manually specifying the dependencies just add one starter 

 you don't need to specify the version number of an artifact. Spring Boot will figure out what version to use – all you need to specify is the version of spring-boot-starter-parent artifact
 
 
 ----
 
 #  Spring Boot Actuator
 
  Actuator is used to expose operational information about the running application
  * Monitoring our app, gathering metrics, understanding traffic, or the state of our database becomes trivial with this dependency.
  
**dependency**
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

* Actuator now shares the security config with the regular App security rules. 
  * Hence, the security model is dramatically simplified.
```
@Bean
public SecurityWebFilterChain securityWebFilterChain(
  ServerHttpSecurity http) {
    return http.authorizeExchange()
      .pathMatchers("/actuator/**").permitAll(). //  by default, all Actuator endpoints are now placed under the /actuator path.
      .anyExchange().authenticated()
      .and().build();
}
```
* Health Indicators: `ReactiveHealthIndicator` has been added to implement reactive health checks.

* Health Groups: we can organize health indicators into groups and apply the same configuration to all the group members. 

* Creating a Custom Endpoint
```
@Component
@Endpoint(id = "features")// The path of our endpoint is determined by
public class FeaturesEndpoint {
 
    private Map<String, Feature> features = new ConcurrentHashMap<>();
 
    @ReadOperation // it'll map to HTTP GET
    public Map<String, Feature> features() {
        return features;
    }
 
    @ReadOperation
    public Feature feature(@Selector String name) {
        return features.get(name);
    }
 
    @WriteOperation // it'll map to HTTP POST
    public void configureFeature(@Selector String name, Feature feature) {
        features.put(name, feature);
    }
 
    @DeleteOperation // it'll map to HTTP DELETE
    public void deleteFeature(@Selector String name) {
        features.remove(name);
    }
 
    public static class Feature {
        private Boolean enabled;
 
        // [...] getters and setters 
    }
 
}
```

*  Extending Existing Endpoints: can easily extend the behavior of a predefined endpoint using the @EndpointExtension annotations

*  Enable All Endpoints: In order to access the actuator endpoints using HTTP
1. `management.endpoints.web.exposure.include=*`. expose all endpoints 
2. `management.endpoint.shutdown.enabled=true`. enable a specific endpoint (for example /shutdown), 
3. `management.endpoints.web.exposure.include=*` `management.endpoints.web.exposure.exclude=loggers`. enabled endpoints except one (for example /loggers),


## Endpoints of Actuator
|name|description|
|----|-----------|
| /health | Shows application health information (a simple ‘status' when accessed over an unauthenticated connection or full message details when authenticated); it's not sensitive by default
| /info | Displays arbitrary application info; not sensitive by default
| /metrics | Shows ‘metrics' information for the current application; it's also sensitive by default
| /trace | Displays trace information (by default the last few HTTP requests)

### customize each endpoint with properties 
* `endpoints.[endpoint name].[property to customize]`
* id – by which this endpoint will be accessed over HTTP
* enabled – if true then it can be accessed otherwise not
* sensitive – if true then need the authorization to show crucial information over HTTP

###  Creating a New Endpoint
> need to have the new endpoint implement the Endpoint<T> interface:
    
```
@Component
public class CustomEndpoint implements Endpoint<List<String>> {
    
    @Override
    public String getId() {
        return "customEndpoint";
    }
 
    @Override
    public boolean isEnabled() {
        return true;
    }
 
    @Override
    public boolean isSensitive() {
        return true;
    }
 
    @Override
    public List<String> invoke() {
        // Custom logic to build the output
        List<String> messages = new ArrayList<String>();
        messages.add("This is message 1");
        messages.add("This is message 2");
        return messages;
    }
}
```
---

# Configure a Spring Boot 

* Change HTTP port number
1. application.properties: `server.port=8083`
2. YAML configuration: 
```
server:
    port: 8083
```
3.  programmatically customize the server port:
```
@SpringBootApplication
public class CustomApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(CustomApplication.class);
        app.setDefaultProperties(Collections
          .singletonMap("server.port", "8083"));
        app.run(args);
    }
}

@Component
public class CustomizationBean implements
  WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {
 
    @Override
    public void customize(ConfigurableServletWebServerFactory container) {
        container.setPort(8083);
    }
}
```
* The Context Path :  
 1. By default, the context path is “/”. 
 2. application.properties:`server.servlet.contextPath=/springbootapp`
 3. YAML-based configuration:
```
server:
    servlet:
        contextPath:/springbootapp
```
4. can be done programmatically
```
@Component
public class CustomizationBean
  implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {
 
    @Override
    public void customize(ConfigurableServletWebServerFactorycontainer) {
        container.setContextPath("/springbootapp");
    }
}
```

*  The White Label Error Page : Spring Boot automatically registers a BasicErrorController bean if you don't specify any custom implementation in the configuration.
```
public class MyCustomErrorController implements ErrorController {
 
    private static final String PATH = "/error";
    
    @GetMapping(value=PATH)
    public String error() {
        return "Error haven";
    }
 
    @Override
    public String getErrorPath() {
        return PATH;
    }
}
```

* Customize the Error Messages: Boot provides /error mappings by default to handle errors in a sensible way.
```
@Component
public class CustomizationBean
  implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {
 
    @Override
    public void customize(ConfigurableServletWebServerFactorycontainer) {        
        container.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
        container.addErrorPages(new ErrorPage("/errorHaven"));
    }
}
```

* Configure the Logging Levels: tune the logging levels in a Boot application in the main properties file
```
logging.level.org.springframework.web: DEBUG
logging.level.org.hibernate: ERROR
```

* Change server : you can exclude the Tomcat dependency and include Jetty or Undertow
```
dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>

@Bean
public JettyEmbeddedServletContainerFactory  jettyEmbeddedServletContainerFactory() {
    JettyEmbeddedServletContainerFactory jettyContainer = 
      new JettyEmbeddedServletContainerFactory();
    
    jettyContainer.setPort(9000);
    jettyContainer.setContextPath("/springbootapp");
    return jettyContainer;
}

```
--- 
