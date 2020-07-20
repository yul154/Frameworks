
@Configuration 
  * serves as a factory for beans that are added to the application context.
  * An auto-configuration is a @Configuration class that is automatically discovered by Spring
  
@ ConditionalOnProperty 
  * tell Spring to only include the EventAutoConfiguration (and all the beans it declares) into the application context if the property eventstarter.enabled is set to true.
  
  
@RestController 
 * combines @Controller and @ResponseBody, two annotations that results in web requests returning data rather than a view.
 
 
@Require
 * a method-level annotation applied to the setter method of a bean property and thus making the setter-injection mandatory
 * This annotation indicates that the required bean property must be injected with a value at the configuration time.
 
@Autowired
* applied on fields, setter methods, and constructors
* injects object dependency implicitly.
* automatic dependency injection.
* default autowiring mode is byType, but can be by-name with @Qualifier

@Qualifier
*  avoid confusion which occurs when you create more than one bean of the same type and want to wire only one of them with a property.

@ Configuration
* used on classes which define beans

@ComponentScan
*

@Bean
* used at the method level
* @Bean annotation works with @Configuration to create Spring beans
* @Bean provide the instantiation implementation for the particular bean
* @Bean is used to declare a single bean, rather than letting Spring do it automatically as in case of Component.

@Component
* @Component is a class level annotation and its purpose it to make the class as spring managed component and auto detectable bean for classpath scanning feature. 
* @Component can be used for , the spring to automatically find the bean and register to the context

@Value
*  a default value expression for the field or parameter to initialize the property with

@Controller
*  can be used to identify controllers for Spring MVC

@Service
* The @Service marks a Java class that performs some service
*  a specialized form of the @Component annotation intended to be used in the service layer.

@Repository
*  used on Java classes which directly access the database
*  marker for any class that fulfills the role of repository or Data Access Object.


# Spring Boot
@EnableAutoConfiguration
*  placed on the main application class.
*  start adding beans based on classpath settings

@SpringBootApplication
