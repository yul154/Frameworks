# Library Management System Using Angular+ Spring Boot+ Spring CRUD+ REST Controller + MySQL

https://letslearnjavanow.wordpress.com/2018/04/16/library-management-system-using-angular-spring-boot-spring-curd-rest-controller-mysql/

## Application.class
```
@SpringBootApplication
public class LmsApplication extends SpringBootServletInitializer {//package our application as a WAR

    public static void main(String[] args) {

        SpringApplication.run(LmsApplication.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(LmsApplication.class);

    }

}

```
* `SpringApplicationBuilder` is an extension of WebApplicationInitializer which runs a SpringApplication from a traditional WAR archive deployed on a web container

* `configure()` uses `SpringApplicationBuilder` to simply register our class as a configuration class of the application

## Configuration layer
```
@EnableSwagger2 // Swagger 2 is enabled through the @EnableSwagger2 annotation.
@Configuration
public class SwaggerConfig {
	
	@Bean
	public Docket userAPI()
	{
		return new Docket(DocumentationType.SWAGGER_12).select().
				apis(RequestHandlerSelectors.basePackage("com.lms.demo.controller")).paths(PathSelectors.regex("/api.*"))
				.build().apiInfo(metaInfo());
	}
	private ApiInfo metaInfo() {

    ApiInfo apiInfo = new ApiInfo(
            "Swagger+ Spring Boot",
            "Swagger API tutorial",
            "1.0",
            "Terms of Service",
            new Contact("Himanshu", "Him2k",
                    "upadhyay.himansh@gmail.com"),
            "Apache License Version 2.0",
            "https://www.apache.org/licesen.html"
    );

    return apiInfo;
}

}
```
* Swagger: an open source project used to generate the REST API documents for RESTful web services.It provides a user interface to access our RESTful web services via the web browser.
* The configuration of `Swagger` mainly centers around the Docket bean.

## Controller Layer
> Designing end points
```
package com.lms.demo.controller;

import com.lms.demo.data.model.Book;
import com.lms.demo.data.model.Order;
import com.lms.demo.data.repository.BookRepository;
import com.lms.demo.data.repository.OrderRepository;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

@RestController
@RequestMapping("/api")
@Api(value = "searchController", description = "This enpoints returns books, makesBooking, cancellation")
public class MyController {

    @Autowired
    private BookRepository bookRepository;
    @Autowired
    private OrderRepository orderRepository;


    @RequestMapping(value = "/getBooks", method = RequestMethod.GET, produces = "application/json")
    @ApiOperation(value = "to get total number of books in library", response = List.class)
    public List<Book> getBooks() {
        List<Book> li = new ArrayList<Book>();
        bookRepository.findAll().forEach(li::add);
        return li;
    }

    @RequestMapping(value = "/getBookingDetails", method = RequestMethod.GET,
            produces = "application/json")
    @ApiOperation(value = "to get total number of booking made", response = List.class)
    public List<Order> getBookingDetails() {
        List<Order> li = new ArrayList<Order>();
        orderRepository.findAll().forEach(li::add);
        return li;
    }


    @RequestMapping(value = "/count", method = RequestMethod.GET, produces = "application/json")
    @ApiOperation(value = "to get count of books", response = Long.class)
    public long countNoofBooks() {
        return bookRepository.count();
    }

    @RequestMapping(value = "/addBook", method = RequestMethod.POST, produces = "application/json")
    @ApiOperation(value = "to add new book in library", response = String.class)
    public void addBooks(@RequestBody List<Book> books) {
        System.out.println(books);
        bookRepository.saveAll(books);


    }

    @RequestMapping(value = "/delBook", method = RequestMethod.POST, produces = "application/json")
    @ApiOperation(value = "to delete book from library", response = String.class)
    public void delBooks(@RequestBody List<Book> books) {
        System.out.println(books);
        bookRepository.deleteAll(books);


    }

    @RequestMapping(value = "/makeBooking", method = RequestMethod.POST,
            produces = "application/json")
    @ApiOperation(value = "to make booking from library", response = String.class)
    public void makeBooking(@RequestBody Order orderDetails) {
        orderRepository.save(orderDetails);


    }

    @RequestMapping(value = "/cancelBooking", method = RequestMethod.POST,
            produces = "application/json")
    @ApiOperation(value = "to cancel booking from library", response = String.class)
    public void cancelBooking(@RequestBody String orderDetails) {
        System.out.println(orderDetails.split(":")[0]);
        orderRepository.deleteByOrderId(orderDetails);


    }

}
```

* `@Api()` :Marks a class as a Swagger resource.
* `@Autowired`:automatic dependency injection


## Persistence Layer
> used CRUD repository provided by Spring for our basic operations.
* `@GenericGenerator`:Using the strategy "foreign" expects one parameter called "property" and the expected value is an entity name. This means your entity's ID will be the same as the linked entity
*  `@Modifying`: used to enhance the @Query annotation to execute not only SELECT queries but also INSERT, UPDATE, DELETE, and even DDL queries.
*  `@Transactional`: Spring creates proxies for all the classes annotated with @Transactional
