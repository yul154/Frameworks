* Properties with Spring and Spring Boot
* @ConfigurationProperties

# Properties with Spring and Spring Boot
> how to set up and use properties in Spring 

Spring
## 1. Register a Properties File via Java Annotations
*  `@PropertySource annotation`: to be used in conjunction with Java-based configuration and the @Configuration annotation

```
@Configuration
@PropertySource("classpath:foo.properties")
public class PropertiesWithJavaConfig {
    //...
}

# using a placeholder to allow you to dynamically select the right file at runtime
@PropertySource({ 
  "classpath:persistence-${envTarget:mysql}.properties"
})
```
* Defining Multiple Property Locations
```
@PropertySource("classpath:foo.properties")
@PropertySource("classpath:bar.properties")
public class PropertiesWithJavaConfig {
    //...
}
```

## 2.Register a Properties File in XML
* new properties files can be made accessible to Spring via the <context:property-placeholder … > namespace element:
```
<context:property-placeholder location="classpath:foo.properties" /> // The foo.properties file should be placed under /src/main/resources
<context:property-placeholder location="classpath:foo.properties, classpath:bar.properties"/>
```

## 3. Using / Injecting Properties

* Injecting a property with the `@Value` annotation
```
@Value( "${jdbc.url}" )
private String jdbcUrl;
```
# obtain the value of a property using the Environment API:
```
@Autowired
private Environment env;

dataSource.setUrl(env.getProperty("jdbc.url"));
```

## Spring boot properties

* `application.properties` – the Default Property File: we don’t have to explicitly register a PropertySource, or even provide a path to a property file.
* `application-environment.properties”` - need to target different environments,will take precedence over the default property file.
* `@TestPropertySource` - `@TestPropertySource("/foo.properties")`, similar effect using the properties argument of the `@SpringBootTest`

---

# @ConfigurationProperties

Spring boot externalized configuration and easy access to properties defined in properties files


* The official documentation advises that we isolate configuration properties into separate POJOs.
```
@Configuration
@ConfigurationProperties(prefix = "mail")// The classpath scanner enabled by @SpringBootApplication finds the `ConfigProperties` class
public class ConfigProperties {
    
    private String hostName;
    private int port;
    private String from;
 
    // standard getters and setters
}

```
* Spring will automatically bind any property defined in our property file that has the prefix mail and the same name as one of the fields in the ConfigProperties class.
```
#Simple properties
mail.hostname=host@mail.com
mail.port=9000
mail.from=mailer@mail.com
```
* On other hand,can use the `@ConfigurationPropertiesScan` annotation to scan custom locations for configuration property classes:
```
@SpringBootApplication
@ConfigurationPropertiesScan("com.baeldung.properties")
public class DemoApplication { 
 
    public static void main(String[] args) {   
        SpringApplication.run(DemoApplication.class, args); 
    } 
}
```
 
* Nested Properties
```
public class Credentials {
    private String authMethod;
    private String username;
    private String password;
 
    // standard getters and setters
}
public class ConfigProperties {
 
    private String host;
    private int port;
    private String from;
    private List<String> defaultRecipients;
    private Map<String, String> additionalHeaders;
    private Credentials credentials;
 
    // standard getters and setters
}
```
* properties file
```
mail.hostname=mailer@mail.com
mail.port=9000
mail.from=mailer@mail.com
 
#List properties
mail.defaultRecipients[0]=admin@mail.com
mail.defaultRecipients[1]=owner@mail.com
 
#Map Properties
mail.additionalHeaders.redelivery=true
mail.additionalHeaders.secure=true
 
#Object properties
mail.credentials.username=john
mail.credentials.password=password
mail.credentials.authMethod=SHA1
```

* `@ConfigurationProperties` on a @`Bean` Method: use @ConfigurationProperties on a @Bean method to bind externalized properties to the Item i
```
@Configuration
public class ConfigProperties {
 
    @Bean
    @ConfigurationProperties(prefix = "item")
    public Item item() {
        return new Item();
    }
}
```
##  Property Validation
* If any of these validations fail, then the main application would fail to start with an IllegalStateException.
```
@Length(max = 4, min = 1)
private String authMethod;

@Min(1025)
@Max(65536)
private int port;

```

## Property Conversion
> conversion for multiple types of binding the properties to their corresponding beans.
1. Duration
```
@ConfigurationProperties(prefix = "conversion")
public class PropertyConversion {
 
    private Duration timeInDefaultUnit;
    private Duration timeInNano;
    ...
}

conversion.timeInDefaultUnit=10
conversion.timeInNano=9ns
```

2. DataSize
```
private DataSize sizeInDefaultUnit;
 
private DataSize sizeInGB;
 
@DataSizeUnit(DataUnit.TERABYTES)
private DataSize sizeInTB;

conversion.sizeInDefaultUnit=300
conversion.sizeInGB=2GB
conversion.sizeInTB=4
```
3. Custom Converter
```
public class Employee {
    private String name;
    private double salary;
}

# a custom converter to convert this property:
conversion.employee=john,2000

# implement the Converter interface,
@Component
@ConfigurationPropertiesBinding
public class EmployeeConverter implements Converter<String, Employee> {
 
    @Override
    public Employee convert(String from) {
        String[] data = from.split(",");
        return new Employee(data[0], Double.parseDouble(data[1]));
    }
}
```

## Immutable @ConfigurationProperties binding
> @ConfigurationProperties-annotated classes may now be immutable.
```
@ConfigurationProperties(prefix = "mail.credentials") //all the fields of ImmutableCredentials are final. Also, there are no setter methods.
@ConstructorBinding
```

