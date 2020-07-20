

why use spring boot(pro) over spring mvc?

* Spring Boot not only provides a lot of convenience by auto-configuration a lot of things for you 
* Simplify annotations, lets you focus only on writing your business logic.
* Provide opinionated 'starter' POMs to simplify your Maven configuration
* Easier customization of application properties
* Embedded Tomcat/Jetty/Undertow support
* The Spring Initializer provides a project generator to make you productive with the certain technology stack from the beginning
* Spring Actuator:  allows seeing inside a running application.
* make your code easier to unit test,@SpringBootTest is used to run unit test on Spring Boot environment.

Spring boot scope?
* No, singleton beans are not thread-safe in Spring framework.

how to change spring scope?
* using spring bean @Scope annotation

Spring boot profile?
* segregate parts of your application configuration and make it be available only in certain environments.
* Any @Component or @Configuration can be marked with @Profile to limit when it is loaded

Important property
* can control logging with Spring Boot by specifying log levels on application.properties

# Test
* In Spring
  * created a JUnit test by flagging the method with a @Test annotation 
  * annotate the JUnit test with a runner to Mockito tests.
  * @Mock to create and inject mocked instances without having to call Mockito.mock manually.
  * @Mock creates a mock. @InjectMocks creates an instance of the class and injects the mocks 
* How to handle exception with junit
  * try catch
  * JUnit RULE
  * annotation `@test`: check only the type of the exception thrown 
* Method to validate your data in JUNIT
  * Bean Validator
* Annotation for junit test
  * @Test: attached can be executed as a test Case.
  * @Before:execute some statement such as preconditions before
  * `@After`:f you want to execute some statements after each Test Case 
  * `@Ignores`: ignore some statements 
  * ``
* One class have database, write junit4, don’t database call(), 
  * testing the DAO (data access object) class,
  * You're testing the syntactic correctness of (generated) SQL.
* Methods to clean()
* How to write junit test case in eclipse
