# Examining Spring Boot essentials

## Old Spring project setup
1. complete with a Maven or Gradle build file including required dependencies.
2. A web.xml file (or a WebApplicationInitializer implementation) that declares Spring’s DispatcherServlet.
3. A Spring configuration that enables Spring MVC.
4. A controller class that will respond to HTTP requests with “Hello World”.
5. A web application server, such as Tomcat, to deploy the application to.

* only one item is specific to developing the functionality (Controller)

## Spring Boot essentials

1. *Automatic configuration* —  automatically provide configuration for application functionality common to many Spring applications.
2. *Starter dependencies* — You tell Spring Boot what kind of functionality you need, and it will ensure that the libraries needed are added to the build.
3. *The command-line interface* — This optional feature of Spring Boot lets you write complete applications with just application code, but no need for a traditional project build.
4. *The Actuator* — Gives you insight into what’s going on inside of a running Spring Boot application.

###  AUTO-CONFIGURATION
* In the previous: written an application that accesses a relational database with JDBC
```
@Bean // JdbcTemplate
public JdbcTemplate jdbcTemplate(DataSource dataSource) {
  return new JdbcTemplate(dataSource);
}

@Bean 
public DataSource dataSource() {
return new EmbeddedDatabaseBuilder() .setType(EmbeddedDatabaseType.H2) .addScripts('schema.sql', 'data.sql') .build();
}
```
* Spring boot
  *  If Spring Boot detects that you have the H2 database library in your application’s class- path, it will automatically configure an embedded H2 database
  

### STARTER DEPENDENCIES
* Spring Boot offers help with project dependency management by way of starter dependencies.
* Starter dependencies are really just special Maven dependencies to aggregate com- monly used libraries under a handful of feature-defined dependencies.
* No longer need to think about what libraries you’ll need to support certain functionality; you simply ask for that functionality by way of the perti- nent starter dependency.
* Spring Boot’s starter dependencies free you from worrying about which versions of these libraries you need.


### THE COMMAND-LINE INTERFACE (CLI)
* you can use if you want to quickly develop a Spring application. It lets you run Groovy scripts, which means that you have a familiar Java-like syntax without so much boilerplate code. 

### THE ACTUATOR
* you can inspect the inner workings of your application, including details such as
1. What beans have been configured in the Spring application context
2.  What decisions were made by Spring Boot’s auto-configuration
3.  What environment variables, system properties, configuration properties, and command-line arguments are available to your application
4. The current state of the threads in and supporting your application
5. A trace of recent HTTP requests handled by your application
6. Various metrics pertaining to memory usage, garbage collection, web requests, and data source usage

---
# Automatic Spring configuration

## `Application` class
```
package readinglist;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication // Enable component-scanning and auto-configuration
public class ReadingListApplication {
Enable component-scanning and auto-configuration
 public static void main(String[] args) { 
        SpringApplication.run(ReadingListApplication.class, args); // Bootstrap the application
} 

}
```
* `SpringBootApplicaton`: combines three other useful annotations
1. `@Configuration` — Designates a class as a configuration class using Spring’s Java-based configuration
2. `@ComponentScan` — Enables component-scanning so that the web controller classes and other components you write will be automatically discovered and registered as beans in the Spring application context.
3. `@EnableAutoConfiguration` - it’s the one line of configuration that enables the magic of Spring Boot auto-configuration.


* If your application requires any additional Spring configuration beyond what Spring Boot auto-configuration provides, it’s usually best to write it into separate `@Configuration` configured classes.

## `ApplicationTests`
```
@SpringBootTest(classes = {EmployeeRepository.class, EmployeeService.class}) // specifies the annotated classes to use for loading an ApplicationContext
class DemoApplicationTests {

	@Test
	void contextLoads() {
	}

}
```
* Uses `SpringBootContextLoader` as the default ContextLoader when no specific @ContextConfiguration(loader=...) is defined.
* Automatically searches for a `@SpringBootConfiguratio`n when nested `@Configuration` is not used, and no explicit classes are specified.
* Registers a TestRestTemplate and/or WebTestClient bean for use in web tests that are using a fully running web server.

## `application.properties`
> CONFIGURING APPLICATION PROPERTIES

## Automatic Configuration

### 1. Focusing on application functionality

*  Define an entity class that represents an object
```
import javax.persistence.Entity;
import javax.persistence.GeneratedValue; import javax.persistence.GenerationType; import javax.persistence.Id;
@Entity // designating it as a JPA entity
public class Book {
  @Id // the member field below is the primary key of current entity
  @GeneratedValue(strategy=GenerationType.AUTO)// specifies that a value will be automatically generated for that field.
  private Long id;
  private String reader;
  private String isbn;
  private String title;
  private String author;
  private String description;
  // getters and setters
```

* Define the repository through which the ReadingList objects will be persisted to the database
```
public interface ReadingListRepository extends JpaRepository<Book, Long> { //the domain type that the repository will work with, and the type of its ID property.
  List<Book> findByReader(String reader);
}
```

* Define Controller


### Conditional configuration
* implement the Condition interface and override its matches() method
```
 public class JdbcTemplateCondition implements Condition {
          @Override
          public boolean matches(ConditionContext context,
                                 AnnotatedTypeMetadata metadata){
                                 // fields
                                 }
```
* Custom condition class
```
@Conditional(JdbcTemplateCondition.class) 
public MyService myService() {} //the MyService bean will only be created if the JdbcTemplateCondition passes.
```


```
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class }) // require that both DataSource and EmbeddedDatabaseType be available on the class- path.
@EnableConfigurationProperties(DataSourceProperties.class) 
@Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class })
public class DataSourceAutoConfiguration { ...
}
```
---
# Customizing configuration

## Overriding Spring Boot auto-configuration
> Creating a custom security configuration

**auto-configuration spring security**
afteradd security starter in xml file, met with an HTTP Basic authentication dialog box

Explicit configuration to override auto-configured security

### 1. writing a configuration class that extends `WebSecurityConfigurerAdapter`.

```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

	@Autowired
	private ReaderRepository readerRepository;
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http 
		.authorizeRequests()
		.antMatchers("/").access("hasRole('READER')") 
		.antMatchers("/**").permitAll()
		.and()
		
		.formLogin()
		.loginPage("/login") // set loginn form path
		.failureUrl("/login?error=true"); 
		}
	@Override
	protected void configure(
	 	AuthenticationManagerBuilder auth) throws Exception {
	auth
	.userDetailsService(new UserDetailsService() { // define custom UserDetailsService
	 @Override
	 public UserDetails loadUserByUsername(String username)
		throws UsernameNotFoundException { 
		return readerRepository.findOne(username);
	} 
	});
 } 
 }
```
* the first `configure()` method specifies that requests for “/” (which ReadingListController’s methods are mapped to) require an authenticated user with the READER role
* The Second `configure()` method set up authenticating users against the database via JPA by `UserDetailsService`

### 2. A repository interface for persisting readers
```
package readinglist;
import org.springframework.data.jpa.repository.JpaRepository;
public interface ReaderRepository
         extends JpaRepository<Reader, String> {
}
```
* There’s no need to write an implementation of Reader- Repository. Because it extends JpaRepository, Spring Data JPA will automatically create an implementation of it at runtime.

### 3. A JPA entity that defines a Reader
```
import javax.persistence.Entity;
import javax.persistence.Id;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority; 
import org.springframework.security.core.userdetails.UserDetails;
@Entity
public class Reader implements UserDetails {
  private static final long serialVersionUID = 1L;
@Id
private String username;
private String fullname;
private String password;
Reader fields
 Licensed to Thomas Snead <n.ordickan@gmail.com>
www.it-ebooks.info
54
CHAPTER 3 Customizing configuration
public String getUsername() {
  return username;
}
public void setUsername(String username) { this.username = username;
}
public String getFullname() {
  return fullname;
}
public void setFullname(String fullname) { this.fullname = fullname;
}
public String getPassword() {
  return password;
}
public void setPassword(String password) { this.password = password;
}
// UserDetails methods
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
return Arrays.asList(new SimpleGrantedAuthority("READER")); }
Grant READER privilege
  @Override
  public boolean isAccountNonExpired() {
    return true;
  }
  @Override
  public boolean isAccountNonLocked() {
    return true;
  }
  @Override
  public boolean isCredentialsNonExpired() {
    return true;
  }
  @Override
  public boolean isEnabled() {
    return true;
  }
}
```
* the `username` field is annotated with `@Id` to designate it as the entity’s ID
* Reader implements the `UserDetails` interface and several of its methods. This makes it possible to use a Reader object to represent a user in Spring Security.

#### Other example
All of this coniguration uses Spring 4.0’s conditional configuration support to make runtime decisions as to whether or not Spring Boot’s configuration should be used or ignored

```
@Bean 
@ConditionalOnMissingBean(JdbcOperations.class) 
public JdbcTemplate jdbcTemplate() {
	return new JdbcTemplate(this.dataSource); 
	}

```
* If there’s already a JdbcOperations bean, then the condition will fail and the jdbcTemplate() bean method will not be used.
* Spring Boot is designed to load application-level configuration before considering its auto-configuration classes.


```
@Configuration
@EnableConfigurationProperties
@ConditionalOnClass({ EnableWebSecurity.class }) @ConditionalOnMissingBean(WebSecurityConfiguration.class) @ConditionalOnWebApplication
public class SpringBootWebSecurityConfiguration {
...
}
```
* the `@EnableWebSecurity` annotation must be available on the classpath.
* `@ConditionalOnWebApplication`, the application must be a web application

## Externalizing configuration with properties
**Spring Boot will draw properties from several property sources**
1. Command-line arguments
2. JNDI attributes from java:comp/env
3. JVM system properties
4. Operating system environment variables
5. Randomly generated values for properties prefixed with random.* (referencedwhen setting other properties, such as `${random.long})`.
6. An application.properties or application.yml file
7. Property sources specified by @PropertySource

### Fine-tuning auto-configuration
* DISABLING TEMPLATE CACHING: Thymeleaf templates are cached by default.
```
spring.thymeleaf.cache=false
```
* CONFIGURING THE EMBEDDED SERVER
```
server.port: 8000
server.ssl.key-store = classpath:keystore.jks
server.ssl.key-store-password = secret
server.ssl.key-password = another-secret
```
* CONFIGURING LOGGING: 
```
logging.path=/var/logs/
logging.file=BookWorm.log  //log entries will be written to /var/logs/BookWorm.log
logging.level.root=WARN logging.level.root.org.springframework.security=DEBUG
```
* CONFIGURING A DATA SOURCE
```
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
pring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
```

### Example
* Externally configuring application beans
```
Controller
@RequestMapping("/")
@ConfigurationProperties(prefix="amazon") // properties injected from “amazon”-prefixed configuration properties
public class ReadingListController {
	  private String associateId;
	  private ReadingListRepository readingListRepository;
	  @Autowired
	  public ReadingListController(ReadingListRepository readingListRepository) { this.readingListRepository = readingListRepository;}
	  public void setAssociateId(String associateId) { 
		this.associateId = associateId;
	}

```

```
amazon.associateId=habuma-20
```

* COLLECTING PROPERTIES IN ONE CLASS : it obtains the information it needs from the injected AmazonProperties bean.
>  it may be better to annotate a separate bean with @ConfigurationProperties and let that bean collect all of the configuration properties.
1. Capturing configuration properties in a bean
```
@Component
@ConfigurationProperties("amazon")
public class AmazonProperties {
  	private String associateId;
	public void setAssociateId(String associateId) { this.associateId = associateId;
}
  	public String getAssociateId() {
    		return associateId;
} }
```
2. Controller injected with properties
```
@Controller
@RequestMapping("/")
public class ReadingListController {
	private ReadingListRepository readingListRepository;
	private AmazonProperties amazonProperties;
	@Autowired
	public ReadingListController(
	      ReadingListRepository readingListRepository,
	      AmazonProperties amazonProperties) { this.readingListRepository = readingListRepository; this.amazonProperties = amazonProperties;
	}
	@RequestMapping(method=RequestMethod.GET)
	public String readersBooks(Reader reader, Model model) {
		List<Book> readingList = readingListRepository.findByReader(reader);
		if (readingList != null) {
			model.addAttribute("books", readingList); 
			model.addAttribute("reader", reader); 
			model.addAttribute("amazonID", amazonProperties.getAssociateId()); // Add Assicaute ID to model
	}
	    return "readingList";
  }
```

* Configuring with profiles
> configure different properties for different deployment environments
```
@Profile("production")
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
...
}
```

```
spring.profiles.active=production,hsqldb//Profiles can be activated by setting the spring.profiles.active 
```

## Customizing application error pages
>  create a custom view that will resolve for a view named “error”.
put it in src/ main/resources/static.

* Any bean that implements Spring’s View interface and has a bean ID of “error” (resolved by Spring’s BeanNameViewResolver)
* A Thymeleaf template named “error.html” if Thymeleaf is configured
* A FreeMarker template named “error.ftl” if FreeMarker is configured
* A Velocity template named “error.vm” if Velocity is configured
* A JSP template named “error.jsp” if using JSP views
---
# Testing with Spring Boot
Spring’s `SpringJUnit4ClassRunner` helps load a Spring application context in JUnit-based application tests.

## Integration testing auto-configuration
* `SpringJUnit4ClassRunner`: a JUnit class runner that loads a Spring application context for use in a JUnit test and enables autowiring of beans into the test class.
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(class = AddressBookConfiguration.class)
public class AddressServiceTests{
     @Autowired
     private AddressService addressSerivce;
     @Test
     public void testService(){
         Address address = addressService.finndByLastName("leo");
	 assertEquals("P", address.getFirstName()); 
	 assertEquals("Sherman", address.getLastName()); 
	 assertEquals("42 Wallaby Way", address.getAddressLine1()); 
	 assertEquals("Sydney", address.getCity()); 
	 assertEquals("New South Wales", address.getState()); 
	 assertEquals("2000", address.getPostCode());
     }
}
```
* `@ContextConfiguration`{even though it helps to load application context, it doesn't enable logging or loads additional properties from application.properties
* `@SpringApplicationConfiguration`:enables logging, the loading of external properties (application.properties or application.yml), and other features of Spring Boot
* `@RunWith` tell JUnit to run using Spring’s testing support.
* `@SpringApplicationConfiguration`:The `SpringRunner` provides support for loading a Spring ApplicationContext


## Testing web applications
>  throw actual HTTP requests at it and assert that it processes those requests correctly

1. Spring Mock MVC — Enables controllers to be tested in a mocked approximation of a servlet container without actually starting an application server
2. Web integration tests — Actually starts the application in an embedded servlet container (such as Tomcat or Jetty), enabling tests that exercise the application in a real application server

### Mocking Spring MVC
> perform HTTP requests against a controller without running the controller within an actual servlet container

To set up a Mock MVC in your test
* use `MockMvcBuilders` with two static methods:`standaloneSetup()` and `webAppContextSetup()`
* `standaloneSetup()` expects you to manually instantiate and inject the controllers you want to test, 
* `webAppContextSetup()` works from an instance of `WebApplicationContext`

```
@RunWith(SpringJUnit4ClassRunner.class) 
@SpringApplicationConfiguration( classes = ReadingListApplication.class) 
@WebAppConfiguration // Enables web context testing
public class MockMvcWebTests {
	@Autowired
	private WebApplicationContext webContext; // Injects WebApplicationContext
	private MockMvc mockMvc;
	@Before
	public void setupMockMvc() {
		mockMvc = MockMvcBuilders 
			.webAppContextSetup(webContext) 
			.build();
}
}
```
* The `setupMockMvc()` method is annotated with JUnit’s `@Before`, indicating that it should be executed before any test methods

```
@Test
public void homePage() throws Exception {
	mockMvc.perform(MockMvcRequestBuilders.get("/readingList")) 
		.andExpect(MockMvcResultMatchers.status().isOk())
		.andExpect(MockMvcResultMatchers.view().name("readingList")) 		
		.andExpect(MockMvcResultMatchers.model().attributeExists("books")) 
		.andExpect(MockMvcResultMatchers.model().attribute("books",
	            Matchers.is(Matchers.empty())));
}

```

### Testing web security
> Apply the Spring Security configurer when creating the MockMvc instance
```
@Before
public void setupMockMvc() {
   mockMvc = MockMvcBuilders
        .webAppContextSetup(webContext) 
	.apply(springSecurity()) 
	.build();
}
```
* The `springSecurity()` method returns a Mock MVC configurer that enables Spring Security for Mock MVC. 

* perform an authenticated request: `@WithMockUser`(Loads the security context with a UserDetails) and `@WithUserDetails`(Loads the security context by looking up a UserDetails)
```
@Test
@WithMockUser(username="craig",
              password="password",
              roles="READER")
public void homePage_authenticatedUser() throws Exception {
...
}
```

## Testing a running application

* `@WebIntegrationTest`:  Spring Boot not only create an application context for your test, but also to start an embedded servlet container. 
* `@WebIntegrationTest(value={"server.port=0"}) (randomPort=true)`: start up the server on a randomly selected port

* `RestTemplate` is fine for simple requests and it’s perfect for testing REST endpoints.
* Selenium:  be able to assert selected content on the page or perform operations which resttemplate can't

---
# Groovy with the Spring Boot CLI

## Developing a Spring Boot CLI application
* When working with the Spring Boot CLI, there is no build specification

### 1. Setting up the CLI project
* Spring Boot CLI projects don’t have a strict project structure
```
$ mkdir readinglist  //  you should create a fresh, clean directory to keep the code separate
$ cd readinglist
$ mkdir static
$ mkdir templates
```
### 2. Eliminating code noise with Groovy
* Groovy doesn’t require qualifiers such as public and private. Nor does it demand semicolons at the end of each line. 
```
class Book {
    Long id
    String reader
    String isbn
    String title
    String author
    String description
}

interface ReadingListRepository {
    List<Book> findByReader(String reader)
    void save(Book book)
}
```
```
@Grab("h2")  //fetch a few dependency libraries on the fly as the application is started.
@Grab("spring-boot-starter-thymeleaf") 
class Grabs {}
```

* The CLI is able to leverage Spring Boot autoconfiguration and starter dependencies.
* The CLI is able to detect when certain types are in use and automatically resolve the appropriate dependency libraries to support those types.
* The CLI knows which packages several commonly used types are in and, if those types are used, adds those packages to Groovy’s default packages.
* By applying both automatic dependency resolution and auto-configuration, the CLI can detect that it’s running a web application and automatically include an embedded web container (Tomcat by default) to serve the application.

## Grabbing dependencies
* `@Grab` is as simple as expressing the dependency coordinates
* `@GrabMetadata`:override the default dependency versions in a properties file.
* `@GrabResolver` annotation enables you to specify additional repositories from which dependencies can be fetched.

---
# Grails in Spring Boot
* Grails is an open source web application framework that uses the Apache Groovy programming language
* GORM Data Services take the work out of implemented service layer logic by adding the ability to automatically implement abstract classes or interfaces using GORM logic. 
* GSP: Creating and testing custom tag libraries using GSP are very easier then JSP since there is no need for developer to code tld files and taglib declarations
## GORM for data persistence
> Grails object-relational mapping

GORM makes database work as simple as declaring the entities that will be persisted. 


```
package readinglist
import grails.persistence.*
@Entity // This is a GORM entity
class Book {
  Reader reader
  String isbn
  String title
  String author
  String description
}
```
```
import org.springframework.beans.factory.annotation.Autowired 
import org.springframework.boot.context.properties.ConfigurationProperties import org.springframework.http.HttpStatus
import org.springframework.stereotype.Controller
import org.springframework.ui.Model
import org.springframework.web.bind.annotation.ExceptionHandler import org.springframework.web.bind.annotation.RequestMapping 
import org.springframework.web.bind.annotation.RequestMethod import org.springframework.web.bind.annotation.ResponseStatus
@Controller
@RequestMapping("/")
@ConfigurationProperties("amazon")
class ReadingListController {
  @Autowired
  AmazonProperties amazonProperties
  @ExceptionHandler(value=RuntimeException.class) 
  @ResponseStatus(value=HttpStatus.BANDWIDTH_LIMIT_EXCEEDED) 
  def error() {
	"error" 
   }
  @RequestMapping(method=RequestMethod.GET)
  def readersBooks(Reader reader, Model model) {
     List<Book> readingList = Book.findAllByReader(reader) 
     model.addAttribute("reader", reader)
     if (readingList) {
        model.addAttribute("books", readingList)
         model.addAttribute("amazonID", amazonProperties.getAssociateId()) }
      "readingList"
  }
  @RequestMapping(method=RequestMethod.POST)
  def addToReadingList(Reader reader, Book book) {
     Book.withTransaction {
        book.setReader(reader) book.save()
  }
    "redirect:/"
  }
}
```
* The most obvious difference between Groovy and java is t it doesn’t work with an injected Repository anymore. Instead, it works directly with the type for persistence.
*  Although we didn’t write a findAllBy- Reader() method , this will work because GORM will implement it for us.
* GORM enabling you to perform persistence directly with the domain and eliminating the need for a repository.

### Defining views with Groovy Server Pages
> The Grails project also offers auto-configuration for Groovy Server Pages (GSP)

* GSP template is sprinkled with expression language references (the parts wrapped in ${})

### Mixing Spring Boot with Grails 3
* Creating a new Grails project
```
$ grails create-app readinglist
```
<img width="346" alt="Screen Shot 2020-07-15 at 12 47 13 PM" src="https://user-images.githubusercontent.com/27160394/87577943-4f3e4b80-c699-11ea-9773-2a5715bb09a0.png">
* grails-app is where you’ll write the controllers, domain types, and other code that makes up the Grails project.
* `apply plugin: "spring-boot"`: able to build and run the Grails application just as you would any other Spring Boot application.

* domain type: it knows where the source files go and it’s also able to generate any related artifacts for us at the same time.
* Grails controller: create-controller	,generate-controller,generate-all
```
$ grails create-controller ReadingList
```
---
# Taking a peek inside with the Actuator

The Actuator offers production-ready features such as monitoring and metrics to Spring Boot applications. 

## Exploring the Actuator’s endpoints

Actuator provide several web endpoints in applicaiont through which you can view internals of you running applicationns
* how beanns are wired together in Spring application context.
* determine what environment properties are available to your application.

<img width="409" alt="Screen Shot 2020-07-16 at 11 57 40 AM" src="https://user-images.githubusercontent.com/27160394/87700188-8d9c3f00-c75b-11ea-9fa8-57fe43b29834.png">

* Actuator dependency
```
<dependency> 
	<groupId>org.springframework.boot</groupId> 
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

```

### configuration endpoints

* `/beans` : 
	1. returns a JSON document describing every single bean in the application context
	2. its Java type, 
	3. any of the other beans it’s injected with
	
* `/autoconfig` : provides a report of all the conditions that are evaluated, grouping them by which conditions passed and which failed

* `/env`: a list of all of the environment properties available to the application, Spring profiles, JVM properties, command-line parameters

* `/configprops`: a report of how those properties are set, whether from injection or otherwise

* `/mappings`:how all of its controllers are mapped to endpoints


### Metrics endpoints
> give a snapshot into an application’s runtime internals.

* `/metrics`: provides a snapshot of various counters and gauges in a running application

* `/trace`:reports details of all web requests, including details such as the request method, path, timestamp, and request and response header

* `/dump`:a snapshot of current thread activity.

* `/health`:MONITORING APPLICATION HEALTH


### Miscellaneous endpoints

* `/shutdown`:In order to shut down your application

* `/info`: reports any information about your application that you might want to expose to callers.


## Connecting to the Actuator remote shell

## Monitoring your application with JMX

The Actuator also exposes its end- points as MBeans to be viewed and managed through JMX (Java Management Exten- sions)


## Customizing the Actuator

the Actuator can be customized
* Renaming endpoints
* Enabling and disabling endpoints
* Defining custom metrics and gauges
* Creating a custom repository for storing trace data
* Plugging in custom health indicators

**Changing endpoint IDs**
```
 endpoints.endpoint-id.id.
```

**Enabling and disabling endpoints**
```
 endpoints.endpointname.enabled = false
```
**Adding custom metrics and gauges**

1. the auto-configuration that enables the Actuator also creates an instance of `CounterService` and `GaugeService `
2. inject the `CounterService` and `GaugeService`instances into any other bean where they’re needed
3. call the methods to update whichever metrics we want.

* `PublicMetrics`: provide as many custom metrics as we want

**Creating a custom trace repository**

problem: the /trace endpoint are stored in an in-memory repository that’s capped at 100 entries
* `InMemoryTraceRepository` bean : set its capacity to some value higher than 100
```
InMemoryTraceRepository traceRepo = new InMemoryTraceRepository(); 
traceRepo.setCapacity(1000);
return traceRepo;
```
* `TraceRepository`: one that finds all stored Trace objects and another that saves a Trace given a Map contain- ing trace information.

**Plugging in custom health indicators**
```
public class AmazonHealth implements HealthIndicator {
}
```

## Securing Actuator endpoints

* the Actuator endpoints can be secured with Spring Security
```
.antMatchers("/shutdown").access("hasRole('ADMIN')")
```
---

# Deploying Spring Boot applications

## Deploying to an application server
### 1. Building a WAR file
* change the <packaging> element’s value from jar to war.
	
```
<packaging>war</packaging>
```
* implementation of Spring’s `WebApplicationInitializer` : create a subclass and override the `configure()` method to specify the Spring configuration class

### 2. Creating a production profile
```
@Bean
@Profile("production")
public DataSource dataSource(){

}
```
* specify that it should only be created if the “production”
