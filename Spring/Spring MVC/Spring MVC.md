# Spring MVC
* POJO based and interface driven
* Based on a dispatcher servlet/ front controller pattern
* Built from the shortcomings of Struts 1
## Terms
* DispatcherServlet: entry/configuration point for the application
* Controller: simple POJOs, handle request and determines wihch view to route to and what model we should get
* RequestMapping: The url and request type that a method is tied to 
* View Resolver: Used to located JSP pages or whatever view we are using
* Servlet-congufL configuration file per dispatcherServlet
* Bean: A spring configured POJO

## MVC
<img width="490" alt="Screen Shot 2019-08-09 at 4 28 47 PM" src="https://user-images.githubusercontent.com/27160394/62807219-d4e96c80-bac2-11e9-89e7-e176d763ca9b.png">

### Tiered Architectures

<img width="388" alt="Screen Shot 2019-08-09 at 4 16 10 PM" src="https://user-images.githubusercontent.com/27160394/62806500-082afc00-bac1-11e9-97ef-14d2f4bf4ff8.png">

### Components
#### 1. Controllers
* Chossing what to do based off an action or a request, and then view is just a result of action
* Interpret user input and transform to input to a model
* Provide access to business logic
* Determines view based off logic
* Annotated with `@Controller`
  1. `@Controller`: tell Spring MVC, this is a controller
  2. `@RequestMapping`: which method is going to handle which request
  3. `@modelAttribute`: binds a method parameter or method return value to a named model attribute and then exposes it to a web view.

#### 2.Views
* How display the results ofwhat our request was to our middle tier
* resolve static files
  1. add resource tag to locate files
  2. add one more `servlet-mapping` for static files type
  3. create static files folder below webcontent(not WEB-INF)
#### 3. Service
* describes the actions of a system
* Annotated with `@Service`
* Where the business logic resides

#### 4. Repository
* Describe the data of a system
* Annotated with `@Repository`
* Focused on persisiting and interacting wit the database
* One-to-one mapping with an object

#### Tags
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
### 2. `HandlerMapping`
* it simply scan the request
* send complete address of controller to `DispatcherServlet`(tell `DispatcherServlet` which controller should get the request)
### 3. `Controller`
* returning a `ModelAndView` object, which contains a model map and a view object; 
### 4. `ViewResolver`
* `DispatcherServlet` consults view resolvers until actual View is determined to render the output
### 5.`View`
* `DispatcherServlet` contacts the chosen view (e.g. Thymeleaf, Freemarker, JSP) with model data and it renders the output depending on the model data
