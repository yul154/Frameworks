# Configuration

## `@Configuration`
* This annotation is used on classes that define beans
* `@Configuration` is an analog for an XML configuration file
## `@EnableWebMvc`
* @EnableWebMvc is equivalent to <mvc:annotation-driven /> in XML. 
* It enables support for @Controller-annotated classes that use @RequestMapping to map incoming requests to a certain method. 
* Only used for Java configuration of Spring MVC web apps
* Convenience annotation for WebMvcConfigurationSupport
* Customizable by extending WebMvcConfigurerAdapter

## `WebApplicationInitializer`
* registers a Spring DispatcherServlet
* creates a Spring web application context
* Replacing Web.xml

## SpringConfig(webconfig)
* register all Spring-related beans 
* using Spring's Java-based configuration style
* Replace Servlet-configu.xml
### `WebMvcConfigurerAdapter`
* replace xml schemas or namespaces
* Handle static resources: `WebMvcConfigurer` 
### I18N
1. Add `getRescourceBundle` method inside of `WebConfig`, Retrieves a Bean
2. `getSessionLocalResovler` bean
3. `addInterceptor` Override from `WebMvcConfigurerAdapter` to look for locale change
4. create `ResourceBundle` messages.properoties
5. Create Spring message tag to display out of messages.properties file
6. add `herf` for changing languages

# Architecture
## Patterns
* MVC
* MVP
* MV*
### ModelView-ViewModel
![httpatomoreillycomsourceoreillyimages1547825](https://user-images.githubusercontent.com/27160394/62961774-6579c280-bdcb-11e9-9a47-fa67033652d7.png)

## Controller
* XML configuration is reduced
* New `RestController` annotation: expose RESTFUL service in an easy way
### `@ComponentScan`
* Override the default scan location for controllers
* Instead of component-scan in xml
```
<context: component-scan base-package="com.sample"># xml
@ComponenScan(basePackages="com.sample")# Java Config
```

## Service
* No `config.xml` require for java configuration
* Annotations for various configuration elements
* Convenience annoations similar to namespaces

## Repository
* No `config.xml` require for java configuration
* Different types of Reposiotories if using Spring Data JPA
* Convertion over Configuration

## View
* Conventions

## Validation
### Customize Annotation
* `@Retention`:You can specify for your custom annotation if it should be available at runtime,
* `@Target`:You can specify which Java elements your custom annotation can be used to annotate.
* `@Documented`:used to signal to the JavaDoc tool that your custom annotation should be visible in the JavaDoc for classes using your custom annotation.
* `@Inherited`: a custom Java annotation used in a class should be inherited by subclasses inheriting from that class

## REST and Ajax
### @RestController
* return type as a ResponseBody 
* No longer need to use the ContentNegotiatingViewResolver
* DispatcherServlet needs to allow access for JOSN/XML request types


# Build
## 1. Set Bean Configuration

```
@Configuration//  indicates that the class declares one or more @Bean methods.
@EnableWebMvc//used to enable Spring MVC
@ComponentScan(basePackages = "packname"): //scan through the given package and register all the Controllers and beans
```
## 2. Configure Spring MVC DispatcherServlet and set up URL mappings to Spring MVC DispatcherServlet.
* declared and mapped for processing all requests either using java or web.xmlconfiguration.
```
public class SpringMvcDispatcherServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer
```
## 3. model class
## 4. Controller Class
