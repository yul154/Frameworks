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
