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

Document Databases
* The main arguments in favor of the document data model are schema flexibility, better performance due to locality,
* A document-like structure(a tree of one-to- many relationships, where typically the entire tree is loaded at once),then it’s probably use a document model.
* cannot refer directly to a nested item within a document
* poor support for joins in document databases
* A document is usually stored as a single continuous string, encoded as JSON, XML, or a binary variant thereof (such as MongoDB’s BSON).
* The locality advantage only applies if you need large parts of the document at the same time.

Convergence of document and relational databases
* Most relational database systems (other than MySQL) have supported XML 


## Query Languages for Data
* SQL is a declarative query language, 
  * In a declarative query language, like SQL or relational algebra, you just specify the pattern of the data you want, what conditions the results must meet 
  * it also hides imple‐ mentation details of the database engine, 
* IMS and CODASYL queried the database using imperative code. 
  * An imperative language tells the computer to perform certain operations in a certain order.
  * stepping through the code line by line, evaluating conditions, updating variables, and deciding whether to go around the loop one more time



**Declarative Queries on the Web**
* The advantages of declarative query languages are not limited to just databases.
* it has more advantage  to locate  elemennt  and make change

**MapReduce Querying**
* MapReduce is neither a declarative query language nor a fully imperative query API but somewhere in between
* the logic of the query is expressed with snippets of code, which are called repeatedly by the processing framework. 
* It is based on the map (also known as collect) and reduce (also known as fold or inject) function
* restriction: They must be pure functions, which means they only use the data that is passed to them as input, they cannot perform additional database queries, and they must not have any side effects. 

### Graph-Like Data Models
*  one-to-many relationships (tree-structured data) or no relationships between records, the document model is appropriate.
*  The relational model can handle simple cases of many-to-many relationships, but as the connections within your data become more complex

A graph consists of two kinds of objects: 
* vertices (also known as nodes or entities) 
* edges (also known as relationships or arcs).

#### Property Graphs
In the property graph model, each vertex consists of:
• A unique identifier
• A set of outgoing edges
• A set of incoming edges
• A collection of properties (key-value pairs)

Each edge consists of:
* A unique identifier
* The vertex at which the edge starts (the tail vertex)
* he vertex at which the edge ends (the head vertex)
* A label to describe the kind of relationship between the two vertices
* A collection of properties (key-value pairs)

Some important aspects of this model
*  Any vertex can have an edge connecting it with any other vertex. There is no schema that restricts which kinds of things can or cannot be associated.
*  Given any vertex, you can efficiently find both its incoming and its outgoing edges, and thus traverse the graph
*  By using different labels for different kinds of relationships, you can store several different kinds of information in a single graph, while still maintaining a clean data model.

**The Cypher Query Language**
*find the names of all the people who emigrated from the United States to Europe.*

> find all the vertices that have a BORN_IN edge to a location within the US, and also a LIVING_IN edge to a location within Europe,
```
Each vertex is given a symbolic name like USA or Idaho, and other parts of the query can use those names to create edges between the vertices

CREATE
  (NAmerica:Location {name:'North America', type:'continent'}),
  (USA:Location      {name:'United States', type:'country'  }),
  (Idaho:Location    {name:'Idaho',         type:'state'    }),
  (Lucy:Person       {name:'Lucy' }),
(Idaho) -[:WITHIN]-> (USA) -[:WITHIN]-> (NAmerica), (Lucy) -[:BORN_IN]-> (Idaho)


MATCH
(person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
(person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (eu:Location {name:'Europe'}) RETURN person.name

```

**Graph Queries in SQL**
*  graph data can be represented in a relational database.
*  but more complex

#### Triple-Stores and SPARQL
In a triple-store, all information is stored in the form of very simple three-part statements: 
* subject
* predicate
* object

The subject of a triple is equivalent to a vertex in a graph. The object is one of two things:
* A value in a primitive datatype, such as a string or a number. In that case, the predicate and object of the triple are equivalent to the key and value of a property on the subject verte
* Another vertex in the graph. In that case, the predicate is an edge in the graph, the subject is the tail vertex, and the object is the head vertex.

**The semantic web**
* publish information as machine-readable data for computers to read
* The Resource Description Framework (RDF)  was intended as a mechanism for different websites to publish data in a consistent format,
 * allowing data from different websites to be automatically combined into a web of data
 * a kind of internet-wide “database of everything

**The RDF data model**
* RDF is also written in an XML format, which does the same thing much more verbosely—see 
* Turtle/N3 is preferable as it is much easier on the eyes

**The SPARQL query language**
* SPARQL is a query language for triple-stores using the RDF data mode

```
Graph Databases Compared to the Network Model
* In CODASYL, a database had a schema that specified which record type could be nested within which other record type. In a graph database, there is no such restriction: any vertex can have an edge to any other vertex. 
* In CODASYL, the only way to reach a particular record was to traverse one of the access paths to it. In a graph database, you can refer directly to any vertex by its unique ID, or you can use an index to find vertices with a particular value.
* In CODASYL, the children of a record were an ordered set, so the database had to maintain that ordering (which had consequences for the storage layout). In a graph database, vertices and edges are not ordered
*  In CODASYL, all queries were imperative, difficult to write and easily broken by changes in the schema. In a graph database, you can write your traversal in imperative code if you want to, but most graph databases also support high-level, declarative query languages such as Cypher or SPARQL
```

### The Foundation: Datalog
*  write it as predicate(subject, object).
*  Datalog is a subset of Prolog
```
name(namerica, 'North America').
type(namerica, continent).
```

## Summary
*  Data started out being represented as one big tree (the hierarchical model), but that wasn’t good for representing many-to-many relationships, so the relational model was invented 
*  Document databases target use cases where data comes in self-contained docu‐ ments and relationships between one document and another are rare
*  Graph databases go in the opposite direction, targeting use cases where anything is potentially related to everything.
*  Document and graph databases have in common is that they typically don’t enforce a schema for the data they store, which can make it easier to adapt applications to changing requirements.
-----
# Storage and Retrieval

the world’s simplest database, implemented as two Bash functions:
```
#!/bin/bash
db_set () {
echo "$1,$2" >> database
}
db_get () {
grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
```
* Every call to `db_set` appends to the end of the file
* Update a key several times, the old versions of the value are not overwritten
  * you need to look at the last occurrence of a key in a file to find the latest valu
* `db_get` has to scan the entire database file from beginning to end, looking for occurrences of the key. In algorithmic terms, the cost of a lookup is O(n)


In order to efficiently find the value for a particular key in the database, we need a different data structure: an index. 
* An index is an additional structure that is derived from the primary data
* This is an important trade-off in storage systems: well-chosen indexes speed up read queries, but every index slows down writes

## Hash Indexes

keep an in-memory hash map where every key is mapped to a byte offset in the data file—the location at which the value can be found

how do we avoid eventually running out of disk space? 
* break the log into segments of a certain size by closing a segment file when it reaches a certain size,
* making subsequent writes to a new segment file.
* perform compaction on these segments,Compaction means throwing away duplicate keys in the log, and keeping only the most recent update for each key

File format
* It’s faster and simpler to use a binary format that first encodes the length of a string in bytes, followed by the raw string

Crash recovery
* If the database is restarted, the in-memory hash maps are lost. 
*  speeds up recovery by storing a snapshot of each segment’s hash map on disk, which can be loaded into mem‐ ory more quickly.

An append-only design turns out to be good fo
*  Appending and segment merging are sequential write operations, which are gen‐ erally much faster than random writes,
*  Concurrency and crash recovery are much simpler if segment files are append- only or immutable
*  Merging old segments avoids the problem of data files getting fragmented over time.


Hash table index also has limitations:
* The hash table must fit in memory, so if you have a very large number of keys, you’re out of luck.
* Range queries are not efficient.

## SSTables and LSM-Trees

Sorted String Table, or SSTable for short that the sequence of key-value pairs is sorted by key.
1. Merging segments is simple and efficient,
2. In order to find a particular key in the file, you no longer need to keep an index of all the keys in memory.

### Constructing and maintaining SSTables

Storage engine work as follows:
 *  When a write comes in, add it to an in-memory balanced tree data structure (forexample, a red-black tree). This in-memory tree is sometimes called a memtable.
 *  When the memtable gets bigger than some threshold—typically a few megabytes —write it out to disk as an SSTable file.
 *  In order to serve a read request, first try to find the key in the memtable, then in the most recent on-disk segment
 *  run a merging and compaction process in the background to combine segment files and to discard overwritten or deleted values.


if the database crashes, the most recent writes (which are in the memtable but not yet written out to disk) are lost. 
 * keep a separate log on disk to which every write is immediately appended,

**Performance optimizations**
```
A Bloom filter is a memory-efficient data structure for approximating the contents of a set. It can tell you if a key does not appear in the database, and thus saves many unnecessary disk reads for nonexistent keys.
```

when looking up keys that do not exist in the database: you have to check the memtable, then the segments all the way back to the oldest (possibly having to read from disk for each one) before you can be sure that the key does not exist. In order to optimize this kind of access, storage engines often use additional Bloom filters 


## B-Trees

In order to make the database resilient to crashes, it is common for B-tree implementations to include an additional data structure on disk: a write-ahead log 
```
(WAL, also known as a redo log). This is an append-only file to which every B-tree modification must be written before it can be applied to the pages of the tree itself. When the data‐ base comes back up after a crash, this log is used to restore the B-tree back to a con‐ sistent state
```
