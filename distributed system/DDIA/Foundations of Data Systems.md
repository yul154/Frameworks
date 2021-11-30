# Foundations of Data Systems
1. Examines what we actually mean by words like reliabil‐ ity, scalability, and maintainability
2. Compares several different data models and query languages
3. Internals of storage engines and looks at how databases lay out data on disk
4. Compares various formats for data encoding
----
# Reliable, Scalable, and Maintainable Applications

**Applications consist of**
*  Store data so that they, or another application, can find it again later (databases)
*  Remember the result of an expensive operation, to speed up reads (caches)
*  Allow users to search data by keyword or filter it in various ways (search indexes)
*  Send a message to another process, to be handled asynchronously (stream processing)
*  Periodically crunch a large amount of accumulated data (batch processing)


## Data Systems
* How do you ensure that the data remains correct and complete, even when things go wrong internally? 
* How do you provide consistently good performance to clients, even when parts of your system are degraded? 
* How do you scale to handle an increase in load? What does a good API for the service look like?

### Reliability
> Continuing to work correctly, even when things go wrong

**Fault-tolerant**
* Things that can go wrong are called faults
* Systems that anticipate faults and can cope with them are called fault-tolerant or resilient
* A fault is not the same as a failure
  * A fault is usually defined as one component of the system deviating from its spec
  * A failure is when the system as a whole stops providing the required service to the user.
* It is impossible to reduce the probability of a fault to zero
  *  prefer tolerating faults over preventing faults
  *  prevention is better than cure（security）


#### Faults
**Hardware Faults**
* redundancy of hardware components was sufficient for most applications

**Software Errors**
* There is no quick solution to the problem of systematic faults in software
* small things can help:
  *  carefully thinking about assumptions and interactions in the system
  *  thorough testing; 
  *  process isolation; 
  *  allowing processes to crash and restart;
  *  measuring, monitoring, and analyzing system behavior in production.

**Human Errors**
* Design systems in a way that minimizes opportunities for error. 
* Decouple the places where people make the most mistakes from the places where they can cause failures.
* Test thoroughly at all levels,
* Allow quick and easy recovery from human errors
* Set up detailed and clear monitoring


### Scalability
> A system’s ability to cope with increased load

**Load**
* Load can be described with a few numbers which we call load parameters.
    * the requests per second to a web server,
    * the ratio of reads to writes in a database,
    * the number of simultaneously active users in a chatroom,
    * the hit rate on a cache, etc.

>Twitter hybrid approach: fanned-out vs. fetched


**Performance**
 Performance :load parameter ->  system resources (CPU, mem‐ ory, network bandwidth) -> performance
 * The response time is process time +  delay time
 * Latency is the duration that a request is waiting to be handled
 * Better use “percentiles” to measure response time: The median is also known as the P50th
 * Queueing delays often account for a large part of the response time at high percentiles. 
 
 **Approaches for Coping with Load**
 * Scaling up (vertical scaling, moving to a more powerful machine) 
 * scaling out (horizontal scaling, distributing the load across multiple smaller machines)
 * shared-nothing architecture: Distributing load across multiple machines
 * elastic: automatically add computing resour‐ ces when they detect a load increase

### Maintainability
Design principles for software system
 * Operability : Make it easy for operations teams to keep the system running smoothly.
 * Simplicity : Make it easy for new engineers to understand the system
  * By removing as much complexity as possible from the system. (Note: this is not the same as simplicity of the user interface.)
  * Making a system simpler DOES NOT necessarily mean reducing its functionality; 
  * removing accidental complexity（as accidental if it is not inherent in the problem that the software solves  but arises only from the implementation）
    *  One of the best tools we have for removing accidental complexity is abstraction.
 * Evolvability: Make it easy to make changes to the system in the future
    * adapting it for unanticipated use cases as requirements change.
      * Agile working patterns provide a framework for adapting to change


# Summary
* Nonfunctional requirements : what it should do, such as allowing data to be stored, retrieved, searched, and processed in various ways)
* functional requirements: general properties like security, reliability, compliance, scalability, compatibil‐ ity, and maintainability
* Reliability means making systems work correctly, even when faults occur
* Scalability means having strategies for keeping performance good, even when load increases.
* Maintainability making life better for the engineering and operations teams who need to work with the system.

----
# Data Models and Query Languages

How is it represented in terms of the next-lower layer?
* Application : model it in terms of objects or data structures. Those structures are often specific to your application.
* Store data: Express them in terms of a general-purpose data model, such as JSON or XML documents, tables in a relational database, or a graph model
* representing that JSON/XML data in terms of bytes in hardware : allow the data to be queried, searched, manipulated, and processed 
* represent bytes in terms of electrical currents, pulses of light, magnetic fields, and more.

| Data model type| Implementation|
|----------------|---------------|
|Relational DB|MYSQL, MS SQL, IBM DB2, Postgre SQL, SQLite|
|Document DB| Cassandra, HBase, Google Spanner, RethinkDb, Mongo  DB|
|Graph Db| Neo4j, Titan, InfiniteGraph, Cypher,AllegroGraph|

## Relational Model 
>  Organized into relations (called tables in SQL), where each relation is an unordered collection of tuples (rows in SQL).
Relational databases
* transaction processing (entering sales or banking trans‐ actions, airline reservations, stock-keeping in warehouses) 
* batch processing (customer invoicing, payroll, reporting).

NoSQL
* A need for greater scalability,including very large datasets or very high write throughput
*  a more dynamic and expressive data model

**The Object-Relational Mismatch**

impedance mismatch : The disconnect between the models is sometimes called an impedance mismatch.

**One-to-many relationship AND Many-to-Many relationship**
* self-contained document: The JSON representation has better locality than the multi-table schema
* Store the text directly, you are duplicating the human-meaningful information in every record that uses i
*    Removing such duplication is the key idea behind normalization in databases.


### Relational Versus Document Databases Today

Cocument Databases
* The main arguments in favor of the document data model are schema flexibility, bet‐ ter performance due to locality,
* A document-like structure(a tree of one-to- many relationships, where typically the entire tree is loaded at once),then it’s probably use a document model.
* cannot refer directly to a nested item within a document
* poor support for joins in document databases
