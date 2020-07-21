# Spring MVC

* POJO based and interface driven
* Based on a dispatcher servlet/ front controller pattern
* Built from the shortcomings of Struts 1

# Terms
* DispatcherServlet: entry/configuration point for the application
* Controller: simple POJOs, handle request and determines wihch view to route to and what model we should get
* RequestMapping: The url and request type that a method is tied to 
* View Resolver: Used to located JSP pages or whatever view we are using
* Servlet-congufL configuration file per dispatcherServlet
* Bean: A spring configured POJO

# MVC
<img width="490" alt="Screen Shot 2019-08-09 at 4 28 47 PM" src="https://user-images.githubusercontent.com/27160394/62807219-d4e96c80-bac2-11e9-89e7-e176d763ca9b.png">

## Tiered Architectures

<img width="388" alt="Screen Shot 2019-08-09 at 4 16 10 PM" src="https://user-images.githubusercontent.com/27160394/62806500-082afc00-bac1-11e9-97ef-14d2f4bf4ff8.png">

## Components

### 1. Controllers
* Chossing what to do based off an action or a request, and then view is just a result of action
* Interpret user input and transform to input to a model
* Provide access to business logic
* Determines view based off logic
* Annotated with `@Controller`
  1. `@Controller`: tell Spring MVC, this is a controller
  2. `@RequestMapping`: which method is going to handle which request
  3. `@modelAttribute`: binds a method parameter or method return value to a named model attribute and then exposes it to a web view.

### 2.Views
* How display the results ofwhat our request was to our middle tier
* resolve static files
  1. add resource tag to locate files
  2. add one more `servlet-mapping` for static files type
  3. create static files folder below webcontent(not WEB-INF)
### 3. Service
* describes the actions of a system
* Annotated with `@Service`
* Where the business logic resides

### 4. Repository
* Describe the data of a system
* Annotated with `@Repository`
* Focused on persisiting and interacting wit the database
* One-to-one mapping with an object

### Tags
> Interact with data in pages
* Types
  1. `spring.tld`：evaluate some errors, set themes and output internationalized messages
  2. `spring-form.tld`: process data, most of tags are mirrors of the html form tags
* Interceptors
  * Able to pre-handle and post-handle web requests
  * Part of the request lifecycle
  * Commonly used for Locale changing

## Workflow in Spring MVC

![SpringMVC](https://user-images.githubusercontent.com/27160394/63177236-7cabf080-c015-11e9-8b66-8b2ff3958e39.png)

### 1. `DispatcherServlet`
* Brower send request to `DispatcherServlet` which can hold all request
* Send request to Controller
* The DispatcherServlet, as any Servlet, needs to be declared and mapped according to the Servlet specification by using Java configuration or in web.xml

```
<servlet>
  	<servlet-name>mvc-dispatcher</servlet-name>
        <servlet-class>
             org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
 	<servlet-name>mvc-dispatcher</servlet-name>
        <url-pattern>*.htm</url-pattern>
  </servlet-mapping>
```

* This class is responsible to notify the Spring framework this our front controller: register the DispatcherServlet and use Java-based Spring configuration.
```
public class FrontControllerConfig extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {

		return new Class[] { WebMvcConfig.class };
	}

	@Override
	protected Class<?>[] getServletConfigClasses() { // load Spring application context.

		return null;
	}

	@Override
	protected String[] getServletMappings() {
		return new String[] { "/" };
	}
}
```

### 2. `HandlerMapping`
* it simply scan the request
* send complete address of controller to `DispatcherServlet`(tell `DispatcherServlet` which controller should get the request)

### 3. `Controller`
* returning a `ModelAndView` object, which contains a model map and a view object; 

```
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.AbstractController;

public class HelloWorldController extends AbstractController{

	@Override
	protected ModelAndView handleRequestInternal(HttpServletRequest request,
		HttpServletResponse response) throws Exception {

		ModelAndView model = new ModelAndView("HelloWorldPage"); 
		model.addObject("msg", "hello world");

		return model;
	}
}
```

* `ModelAndView`
1. Represents a model and view returned by a handler, to be resolved by a DispatcherServlet. 
2. The view can take the form of a String view name which will need to be resolved by a ViewResolver object; 
3. The model is a Map, allowing the use of multiple objects keyed by name.

### 4. `ViewResolver`
* The viewResolver will find the file with following mechanism : prefix + view name + suffix, which is /WEB-INF/pages/HelloWorldPage.jsp.
* `DispatcherServlet` consults view resolvers until actual View is determined to render the output
```
<bean id="viewResolver"
    	class="org.springframework.web.servlet.view.InternalResourceViewResolver" >
        <property name="prefix">
            <value>/WEB-INF/view/</value>
        </property>
        <property name="suffix">
            <value>.jsp</value>
        </property>
    </bean>
```
* Java-based configuration: 
```
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = { "com.howtodoinjava.demo.spring"})
public class WebMvcConfig implements WebMvcConfigurer { //customizing or adding to the default Spring MVC configuration enabled through the use of @EnableWebMvc.
 
   @Bean
   public InternalResourceViewResolver resolver() {
      InternalResourceViewResolver resolver = new InternalResourceViewResolver();
      resolver.setViewClass(JstlView.class);
      resolver.setPrefix("/WEB-INF/view/");
      resolver.setSuffix(".jsp");
      return resolver;
   }
 
}
```

### 5.`View`
* `DispatcherServlet` contacts the chosen view (e.g. Thymeleaf, Freemarker, JSP) with model data and it renders the output depending on the model data
