# ORM,JPA,Hibernate

* JPA(interface): JPA is the standard specification for ORM in Java EE. a Java specification for accessing, persisting, and managing data between Java objects / classes and a relational database.

* ORM: Object Relational Mapping  a concept/process of converting the data from Object oriented language to relational DB
* Hibernate(implmentation) is an implementation of JPA and uses ORM technique.

# Hibernate
> It is an open source, lightweight, ORM (Object Relational Mapping) tool. 

* Hibernate implements the specifications of JPA (Java Persistence API) for data persistence.

## Core concept
Session Factory : Any user application requests Session Factory for a session object.
* Session Factory uses configuration information from above listed files, to instantiates the session object appropriately.

Session : used to get a physical connection with a database. 
* This is represented by the org.hibernate.Session class. 
* The instance of a session can be retrieved from the SessionFactory bean.

Query : It allows applications to query the database for one or more stored objects. Hibernate provides different techniques to query database, including NamedQuery and Criteria API.

Transaction :Transaction is a single-threaded, short-lived object used by the application to specify atomic units of work.

Persistent objects : These are plain old Java objects (POJOs), which get persisted as one of the rows in the related table in the database by hibernate.They can be configured in configurations files (hibernate.cfg.xml or hibernate.properties) or annotated with @Entity annotation.

## Hibernate Caching
* First Level Caching is the default caching in hibernate. It executes query at the end of the transaction to reduce the interaction with the DB. It is associated with the session object.

* Second Level Caching is associated with the SessionFacory object. It loads the entity objects at the SessionFacory level while executing a transaction. As a result, those objects are available to the entire application and do not bound to a single user.

Second level caching is configured in hibernate.cfg.xml file. Following are the popular open source cache implementations

# What are the important benefits of using Hibernate Framework?
* Hibernate cache helps us in getting better performance.
* Hibernate eliminates all the boiler-plate code that comes with JDBC and takes care of managing resources, so we can focus on business logic.
* Hibernate is easy to integrate with other Java EE frameworks, it’s so popular that Spring Framework provides built-in support for integrating hibernate with Spring applications.
* Hibernate implicitly provides transaction management,
* Hibernate Query Language (HQL) is more object oriented and close to java programming language.

# How define JPA and how implement JPA

* In the  JPA, each Entity class will correspond to one table in the database. There are very many tables in the database, therefore, there will be a lot of entity classes. You frequently have to work with  Entities, and write  DAO classes ( Data Access Object) to manipulate data through these Entities
* Define an extended interface- Repository<T,ID> interface, and declare methods to manipulate with the data of this Entity. 
