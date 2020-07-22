# JPA
* ORM is the approach of taking object-oriented data and mapping to a relational Database.
* Hibernate is an implementation of JPA and uses ORM technique.
* JPA is the EE standard specification for ORM in Java EE
* POJO based
* use the standard JPA APIs and configure your application to use Hibernate as the provider of the spec under the covers.

# Java Data Access Layer
## 1. JDBC
* simple database with few tables
* native SQL needs
## 2. JEE Batch or Hadoop
* Needs to perform a lot of SQL operations
* Needs to create object graph in memory
## 3. ORM JPA/Hibernate
* Data graphs without excessive relationshipe
## Nosql MongoDB
* data is topical in nature 
* Thread relational data
* can be stored in a standard relational database

--------------
# JPA
## Spring Repositories
> data access contracts that client code can depend and bind to 
* Client service code don't even notice when switch the interface implementation out
* `@Repository` annotation:marks the specific class as a Data Access Object
* Entities map one to one with a JPA repository
* indicate that the class provides the mechanism for storage, retrieval, search, update and delete operation on objects.

![guide-spring-data-jpa-3](https://user-images.githubusercontent.com/27160394/64201580-06085300-ce5d-11e9-85b0-60bef64dc3bf.png)

## CrudRepository
* provides sophisticated CRUD functionality for the entity class that is being managed.
## `PagingAndSortingRepository`
* Extension of CrudRepository to provide additional methods to retrieve entities using the pagination and sorting abstraction.
## JpaRepository
* provides some JPA-related methods such as flushing the persistence context and deleting records in a batch
# Query DSL
* is an extensive Java framework, which allows for the generation of type-safe queries in a syntax similar to SQL
----
# Spring Data JPA
Spring Data takes this simplification one step forward and makes it possible to remove the DAO implementations entirely
*  by implementing one of the Repository interfaces, the DAO will already have some basic CRUD methods (and queries) defined and implemented.

```
public interface JpaRepository<T,ID>
extends PagingAndSortingRepository<T,ID>, QueryByExampleExecutor<T>{

}
```
* First, by extending JpaRepository we get a bunch of generic CRUD methods into our type that allows saving
* Second,this will allow the Spring Data JPA repository infrastructure to scan the classpath for this interface and create a Spring bean for it.

## Query methods
* By default Spring Data JPA will automatically parses the method name and creates a query from it. The query is implemented using the JPA criteria API.

----
# Example

## 1. Define a Simple Entity
```
@Entity
public class Customer {

  @Id
  @GeneratedValue(strategy=GenerationType.AUTO)
  private Long id;
  private String firstName;
  private String lastName;

  protected Customer() {}

  public Customer(String firstName, String lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  @Override
  public String toString() {
    return String.format(
        "Customer[id=%d, firstName='%s', lastName='%s']",
        id, firstName, lastName);
  }

  public Long getId() {
    return id;
  }

  public String getFirstName() {
    return firstName;
  }

  public String getLastName() {
    return lastName;
  }
}
```

## 2. Create Simple Queries
* using JPA to store data in a relational database
```
package com.example.accessingdatajpa;

import java.util.List;

import org.springframework.data.repository.CrudRepository;

public interface CustomerRepository extends CrudRepository<Customer, Long> {

  List<Customer> findByLastName(String lastName);

  Customer findById(long id);
}
```
* Spring Data JPA also lets you define other query methods by declaring their method signature
* Spring Data JPA need not write an implementation of the repository interface. Spring Data JPA creates an implementation when you run the application.
  Customer findById(long id);

## 3. Create an Application Class
```

@SpringBootApplication
public class AccessingDataJpaApplication {

  private static final Logger log = LoggerFactory.getLogger(AccessingDataJpaApplication.class);

  public static void main(String[] args) {
    SpringApplication.run(AccessingDataJpaApplication.class);
  }

  @Bean
  public CommandLineRunner demo(CustomerRepository repository) {
    return (args) -> {
      // save a few customers
      repository.save(new Customer("Jack", "Bauer"));
      repository.save(new Customer("Chloe", "O'Brian"));
      repository.save(new Customer("Kim", "Bauer"));
      repository.save(new Customer("David", "Palmer"));
      repository.save(new Customer("Michelle", "Dessler"));

      // fetch all customers
      log.info("Customers found with findAll():");
      log.info("-------------------------------");
      for (Customer customer : repository.findAll()) {
        log.info(customer.toString());
      }
      log.info("");

      // fetch an individual customer by ID
      Customer customer = repository.findById(1L);
      log.info("Customer found with findById(1L):");
      log.info("--------------------------------");
      log.info(customer.toString());
      log.info("");

      // fetch customers by last name
      log.info("Customer found with findByLastName('Bauer'):");
      log.info("--------------------------------------------");
      repository.findByLastName("Bauer").forEach(bauer -> {
        log.info(bauer.toString());
      });
      // for (Customer bauer : repository.findByLastName("Bauer")) {
      //  log.info(bauer.toString());
      // }
      log.info("");
    };
  }

}
```
* To get output (to the console, in this example), you need to set up a logger
* `CommandLineRunner`interfaces: used to indicate that a bean should run when it is contained within a SpringApplication
