**Spring Initializr**: offers a fast way to pull in all the dependencies you need for an application and does a lot of the setup for you

**Gradle and maven**:Both are able to cache dependencies locally and download them in parallel. As a library consumer, Maven allows one to override a dependency, but only by version. Gradle provides customizable dependency selection and substitution rules that can be declared once and handle unwanted dependencies project-wide.

```
@RestController
public class HelloController {

	@RequestMapping("/")
	public String index() {
		return "Greetings from Spring Boot!";
	}

}
```
The class is flagged as a `@RestController`, meaning it is ready for use by Spring MVC to handle web requests. 
`@RequestMapping`:
* maps `/`to the index() method. When invoked from a browser or by using curl on the command line, the method returns pure text. That is because @RestController combines @Controller and @ResponseBody, two annotations that results in web requests returning data rather than a view.
* `@GetMapping` is a composed annotation that acts as a shortcut for @RequestMapping(method = RequestMethod.GET).

```
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

	@Bean
	public CommandLineRunner commandLineRunner(ApplicationContext ctx) {
		return args -> {

			System.out.println("Let's inspect the beans provided by Spring Boot:");

			String[] beanNames = ctx.getBeanDefinitionNames();
			Arrays.sort(beanNames);
			for (String beanName : beanNames) {
				System.out.println(beanName);
			}

		};
	}

}
```
**@SpringBootApplication** is a convenience annotation that adds all of the following:

* `@Configuration`: Tags the class as a source of bean definitions for the application context.
* `@EnableAutoConfiguration`: Tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings. For example, if spring-webmvc is on the classpath, this annotation flags the application as a web application and activates key behaviors, such as setting up a DispatcherServlet.
* `@ComponentScan`: Tells Spring to look for other components, configurations, and services in the com/example package, letting it find the controllers.


**@Bean**: the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container.

**CommandLineRunner** is an interface used to indicate that a bean should run when it is contained within a SpringApplication. A Spring Boot application can have multiple beans implementing CommandLineRunner
