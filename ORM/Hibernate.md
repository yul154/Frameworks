# ORM
> object relation mapping
* declare how java class mapped to relational tables 
* ORM deal with SQL operations, automatically translate to real table

# Hibernate
> A framework for mapping a object-oriented domain model to a relational database

* Hibernate implemented the JPA specification but it also does more than JPA
* Mapping Java classes to database tables is accomplished through the configuration of an XML file or by using Java annotations.

## Functions
* Specify the configuration to use 
* Connect database
* Specify the way which out java object should be mapped to relaional tables
* Create SQL 

## With JPA(Java Persistence API) 
* JPA is an abstract layer, can be treat as interface, and Hibernate can be treat as java class
* Persisting Java objects in a SQL database
* JPA stands for Java Persistence API. It defines a persistence model for object-relational mapping. This is a Java language specification and it lets us map, store, update, and retrieve from relational databases to Java Objects and vice versa

## Architecture
![architecture](https://user-images.githubusercontent.com/27160394/61223047-e7ea6600-a6e9-11e9-96a3-156d83d2cdb3.jpg)

* Persistent Object lives both in applications code an in Hibernate
* Hibernate interface with database through JDBC
* A configuration file can tell Hibernate 
  1. what JBDC driver to use
  2. what dialect to speak to the SQL database
* A mapping files: how to map Java object to relational tables

### Hibernate Box
<img width="576" alt="Screen Shot 2019-07-15 at 10 25 20 AM" src="https://user-images.githubusercontent.com/27160394/61223571-ebcab800-a6ea-11e9-8846-4f7d4bdbc024.png">

* Configuration class: read configuration file and setup Hibernate(create class instance)
* Session factory class : create a single class instance to manage sessions, ask session factory for the session.
* Session: used to get a physical connection with a database. 
  1. firstly ask session to start a new transaction, when it done, and commit transaction
  2. do queries and commands
  3. commit the transaction
* Transaction - A transaction represents a single unit of database work. This is an optional object.
* Query – Represents SQL or HQL queries to retrieve or modify data. The query object is used to bind the parameters

### Why Hibernate
* accessing the database by using JDBC. Programmers often write complex SQL queries and map the result to Java objects programmatically. 
* This made application tightly coupled and made it hard to port the application to a different database as SQL syntaxes vary between the databases
* we can easily map Java objects to database tables either using XML configuration or annotations.
* database independency - Hibernate abstracts the SQL queries using higher-level Hibernate Query Language, this allows us to write the same queries independent of the database independently.

### Hibernate pros and cons

Pros
* Hibernate uses its own query language HQL and it lets us write queries in a database-independent manner
* Lets us connect Java Classes to Database tables using XML configuration or using annotation
* Hibernate has the ability to cache the results to optimize the read performance
* It supports object inheritance, storing collections to databases

Cons
* Hibernate is slightly less performant compared to JDBC as it has to convert the HQL to its native SQL each time. It runs many SQL queries in the backend based on our object mapping.
* Hibernate doesn’t allow us to insert multiple records into the same table using a single query
* Complex data fetches might lead to multiple iterations of the object-to-table mapping

# Set up 
## 1.add dependencies for Hibernate and MySQL Connector Java libraries.
```
<dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
         <version>6.0.5</version>
      </dependency>
      <!-- Hibernate 5.2.6 Final -->
      <dependency>
         <groupId>org.hibernate</groupId>
         <artifactId>hibernate-core</artifactId>
         <version>5.2.6.Final</version>
      </dependency>
```
## 2. creating Hibernate configuration file (hibernate.cfg.xml) 
* tell Hibernates how to connect to the database 
```
<session-factory >
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/protein_tracker?serverTimezone=UTC</property>
        <property name="hibernate.connection.username">student</property>
        <property name="hibernate.connection.password">student</property>
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.default_schema">protein_tracker</property>
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</property>
</session-factory>
```
* which Java classes should be mapped to database tables.

### 3. Loading Hibenrate Session Factory
1. load cfg.xml
```
Configuration configuration= new Configuration().configure();
```
2. Open a service registry:  hosts and manages Services
```
serviceRegistry = new StandardServiceRegistryBuilder().applySettings(configuration .getProperties()).build();
````
3. Build a `SessionFactory`
```			
sessionFactory=configuration.buildSessionFactory(serviceRegistry);
````

# Mapping 
<img width="929" alt="Screen Shot 2019-07-16 at 9 32 25 AM" src="https://user-images.githubusercontent.com/27160394/61298667-a2db3800-a7ac-11e9-8041-f0a6e1fd61d3.png">

## Save data
```
session.beginTransaction();// open a transaction
User user =new User();
user.setName("ll");
user.setGoal(250);
session.save(user);// save data in session
session.getTransaction().commit();// obtain transaction object and tell the database to save all the changes in the current transaction.
```
## Automatic Schema Creation
#### `hbm2dll.auto`
* Turn on `hbm2dll.auto`option in config file, which allow auto generation of the ddl
* There is no guarantee your existing data won't be harmed
* Options
  1. validate: validate schema matches the mapping file,make no change to the database
  2. update:no drop existing table, will update or add new one when they missing
  3. create: creates the schema, destroying previous data.
  4. create-Drop: drop the schema when the SessionFactory is closed explicitly, typically when the application is stopped.

## Retrieve data out
```
session.beginTransaction();
User NewUser=(User) session.get(User.class, 1); //User NewUser=(User) session.load(User.class, 1);
System.out.println(NewUser.getName());
System.out.println(NewUser.getGoal());
session.getTransaction().commit();
```

`load()`|`get()`
--------|-----------
will always return a “proxy” (proxy is an object with the given identifier value)|return the real object, an object that represent the database row, not proxy.
without hitting the database|It always hit the database
If no row is found, a ObjectNotFoundException will throw.|It will always return null , if the identity value is not found in database.
 
 #### automatically update object
 ```
 NewUser.setTotal(NewUser.getTotal()+50000);
 ```
* Database would automatically persist changes to those objects after manipulate object in code



# Mapping relationships
* Value types: If an object don’t have its own database identity,never be referenced, belong to another piece of data
* entity type: If an object has its own database identity (primary key value,other entity can refer to it 
* Directionality: relationship can be unidirectional or bidirectional
* Components: Object is composed of some other object that were treating as a value type
### Value Type Collection
> map a single database value (column) to a single, non-aggregated Java type.
* Value type collections are mapped to two tables

### mapping file
* add one component
```
<component name="proteinData">
	<property name="total" type="int">
			<column name="TOTAL" />
	</property>
	<property name="goal" type="int">
	    <column name="GOAL" />
	</property>
</component>
```
* The Hibernate mapping element used for mapping a collection depends upon the type of interface. For example, a <set> element is used for mapping properties of type Set.
* A list of components
	
```
<composite-element class="com.practice.proigrammer.UserHistory">
	<property name="entrytime" type="date" column="ENTRY_TIME" />
	<property name="entry" type="string" column="ENTRY" />
</composite-element>
```

### mapping entity relationships
#### One to many
* Avoid update twice:`inverse="true"`
* If a ‘Stock’ is saved, all its referenced ‘stockDailyRecords’ should be saved into database as well--->`casecade="save-update"`
```
<list name="history" table="User_his" inverse="true" cascade="save-update">
	<key column="USER" />
	<list-index column="Position" />
	<one-to-many class="com.practice.proigrammer.UserHistory" />
</list>
```

#### One to one 
* One to one data
   1. one side is value type, we call it component
   2. they are entity type, it is one to one maping
* The `<generator>` class is a sub-element of id, used to generate the unique identifier for the objects of persistent class
```
 <one-to-one name="proteinData" class="com.practice.proigrammer.ProteinData" cascade="save-update" />

```
#### Join
```
<join table="User_GoalAlert" optional="true">
	<key column="User"/>
	<many-to-one name="goalAlert" 
	column="GoalAlert_ID"
	not-null="true"
	unique="true" cascade="save-update"/>
</join>
```
#### Many-to-many
```
<set name="goalAlerts" table="User_GoalAlert" cascade="save-update">
	<key column="User"/>
	<many-to-many class="com.practice.proigrammer.GoalAlert column= "GoalAlert_ID" />
</set>
```

## QUERY
* HQL/JPA
* Criteria API
* Native SQL

### HQL
* It is operating on java objects

#### Criteria API
* Criteria is API  for retrieving entities by composing Criterion objects( `where` clause)
* use *restrictions* in your queries to selectively retrieve objects;
* can apply different kind of filtration rules and logical conditions
* Projection:`max()`,`avg()`,`sum()`

```
session.createCriteria(User.class)
.createAlias("proteinData","pd")#join
.add(Restrictions.or(
 Restriction.eq("name","Joe")).
 setProjection(Projections.property("pd.total")))#join
```
* QBE(query by example)
```
User user=new User();
user.setName("joe")
Example e= Example.create(user).ignoreCase();
session.createCriteria(User.class).add(e)
```
# Advance features
## Caching
* first level cache is 
  1. associated with the Session object(default). 
  2. worked automatically
  3. When we work with a session, don't immediately send SQL statements to the database after each operation. Instead the objects we are creating and changing are cached in the first level cache and all of the changes to the database are persisted at once. 
* Second level caching: use cache provider(Ehcache)
<img width="333" alt="Screen Shot 2019-10-14 at 12 46 50 PM" src="https://user-images.githubusercontent.com/27160394/66768524-bfa50b80-ee80-11e9-96c3-c289ed1b62cf.png">

### Interceptor
* intercept entity operations like save or update
### Listener
> Monitor specific event that occurs and respond to it.
* can be use either in Interceptor or replace it 
### Filter
* 


# Summary
## Why did you choise Hibernate, What is Hibernate + how it’s different with JDBC
* A framework for mapping a object-oriented domain model to a relational database
* No knowledge of SQL is needed because Hibernate is a set of objects and a table is treated as an object, 
* Hibernate is database independent: You can work with any database you want
* HQL is fully object-oriented and more close to the Java programming language 
* Hibernate supports lazy loading,It enhances the performance of the application.
* Hibernate supports cacheing mechanism.Hibernate retains the objects in cache to reduce repeated hits to the database.
* It is easy to create an association between the tables using Hibernate than JDBC.

## What is ORM + What is Data Persistence
* declare how java class mapped to relational tables 
* Persistence basically means saving object state in memory for future reference.`
## What is Lazy/Eager Loading + advantage/disadvantage of each
* Lazy load:  it does not actually load all dependent objects when loading current one . Instead, it loads them when requested to do so
* Lazy loading in hibernate improves the performance. It loads the child objects on demand
* you have a parent objects and that parent has a collection of children. Hibernate now can "lazy-load" the children, which means that it does not actually load all the children when loading the parent. Instead, it loads them when requested to do so. 
* "Lazy loading" means that an entity will be loaded only when you actually accesses the entity for the first time.
*  @OneToMany(fetch = FetchType.LAZY, mappedBy = "user")
*  Hibernate intercepts calls to the entity by substituting a proxy for it derived from the entity’s class.
* Lazy loading tends to be more useful when large collections are involved.
* If you are going to always use something (for sure), you can eager load it.
* LAZY loading is used in cases where the related entity size is huge and it's not required to be fetched every time on the other hand


## Types of cache of Hibernate + caching levels
* First Level Cache: Hibernate first level cache is associated with the Session object. Hibernate first level cache is enabled by default and there is no way to disable it. However hibernate provides methods through which we can delete selected objects from the cache or clear the cache completely.
* Second Level Cache: second-level cache is SessionFactory-scoped, meaning it is shared by all sessions created with the same session factory. is disabled by default but we can enable it through configuration.
* Query-level is an optional cache for query result sets.

## What is session factory and session
* SessionFactory is a factory class for Session objects..
* A Session is used to get a physical connection with a database. 
## Different between hibernate 4 and hibernate 5
* Redesign SessionFactory building
* Introduces a ServiceRegistry.
## different between load and get
* load will return a proxy object.get will return a actual object, and returns null if it wont find any object.
* So that the load() method gets better performance because it support lazy loading.
## Annotations you use in Hibernate
* `@Entity`: defines that a class can be mapped to a table
* `@Table`: allowing you to override the name of the table
* `@Id and @GeneratedValue `: @Id for primary key, @GeneratedValue  generation strategy to be used 
* `@Column`:

## What are the states of the object in hibernate?


* Transient: The object is in a transient state if it is just created but has no primary key (identifier) and not associated with a session.
* Persistent: The object is in a persistent state if a session is open, and you just saved the instance in the database or retrieved the instance from the database.
* Detached: The object is in a detached state if a session is closed. After detached state, the object comes to persistent state if you call lock() or update() method.
