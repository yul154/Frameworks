# What is REST API?
 * Representational State Transfer a.k.a REST is an architectural style as well as an approach for communications purpose that is often used in various web services development.
 * It defines a set of principles that a system must comply with in order to get some benefits such as loose coupling between the client and the server, scalability, reliability, and better performance.
 * In REST architecture, a REST Server simply provides access to resources and REST client accesses and modifies the resources. Here each resource is identified by URIs/ global IDs. 
 * REST uses various representation to represent a resource like text, JSON, XML. JSON is the most popular one.
 
In most cases, the design of a so-called RESTful API consists of:
 * defining the resources accessible via HTTP
 * identifying such resources with URLs
 * mapping the CRUD (Create, Retrieve, Update, Delete) operations on these resources to the standard HTTP methods (POST, GET, PUT, DELETE)
 
# Principles of REST API
 
## 1. Stateless
* In REST applications, each request must contain all of the information necessary to be understood by the server, rather than be dependent on the server remembering prior requests.
* The server does not store any state about the client session on the server-side.
* Statelessness enables greater scalability since the server does not have to maintain, update or communicate that session state

## 2.Uniform Interface
The uniform interface constraint defines the interface between clients and servers. It simplifies and decouples the architecture, which enables each part to evolve independently. 

### The four guiding principles of the uniform interface are:
1. Resource-Based：Individual resources are defined in requests using URIs as resource identifiers and are separate from the responses that are returned to the client.
2. Manipulation of Resources Through Representations：when a client holds a representation of a resource, including any metadata attached, it has enough information to modify or delete the resource on the server, provided it has permission to do so.
3. Self-descriptive Messages: Each message includes enough information to describe how to process the message
4. Hypermedia as the engine of application state: Clients deliver state via body contents, query-string parameters, request headers and the requested URI (the resource name). Services deliver state to clients via body content, response codes, and response headers

## 3.Client-Server
* The uniform interface separates clients from servers. 
* This separation of concerns means that, for example, clients are not concerned with data storage, which remains internal to each server, so that the portability of client code is improved.
* Servers are not concerned with the user interface or user state, so that servers can be simpler and more scalable. 
* Servers and clients may also be replaced and developed independently, as long as the interface is not altered.

## 4.Cacheable
* As the clients can cache responses, they need to be implicitly or explicitly defined as cacheable or not to prevent clients from reusing state or inappropriate data in response to further requests. 
* Well-managed caching partially or completely eliminates some client–server interactions and improves the performance.

## 5. Layered System
* The layered system architecture allows an application to be more stable by limiting component behavior.  
* This type of architecture helps in enhancing the application’s security as components in each layer cannot interact beyond the next immediate layer they are in. 
----
# REST API building blocks

## Resources (URIs)
* Resources should describe the “objects” that your API describes, with an identifier that uniquely identifies each object.
* REST architecture treats every content as a resource. These resources can be Text Files, Html Pages, Images, Videos or Dynamic Business Data.
* A resource in REST is a similar Object in Object Oriented Programming or is like an Entity in a Database
* Each resource is identified by URIs/ Global IDs.
* REST uses various representations to represent a resource where Text, JSON, XML

## Messages/ Resource Methods

* RESTful Web Services make use of HTTP protocols as a medium of communication between client and server.
* Messaging: A client sends a message in form of a HTTP Request and the server responds in the form of an HTTP Response

### HTTP Request/Response

Http Request

1. Verb − Indicates the HTTP methods such as GET, POST, DELETE, PUT, etc.
2. URI − Uniform Resource Identifier (URI) to identify the resource on the server.
3. HTTP Version − Indicates the HTTP version.
4. Request Header − Contains metadata for the HTTP Request message as key-value pairs.
5. Request Body − Message content or Resource representation.

HTTP Response
* Status/Response Code − Indicates the Server status for the requested resource.
* HTTP Version − Indicates the HTTP version
* Response Header − Contains metadata for the HTTP Response message as keyvalue pairs - content length, content type, response date, server type, etc.
* Response Body − Response message content or Resource representation.


### Resource Methods
RESTful web service makes heavy uses of HTTP verbs to determine the operation to be carried out on the specified resource(s).

| Http verb| CRUD| Description|
|----------|-----|------------|
| POST|CREATE|The POST verb is most-often utilized to **create** new resources. POST is neither safe nor idempotent.|
|GET|READ|The HTTP GET method is used to **read** (or retrieve) a representation of a resource，GET returns a representation in XML or JSON and an HTTP response code of 200 (OK)|
|PUT|UPDATE/REPLACE|most-often utilized for **update** capabilities, if you create or update a resource using PUT and then make that same call again, the resource is still there and still has the same state as it did with the first call.
|PATCH|UPDATE/MODIFY|used for **modify** capabilities.The PATCH request only needs to contain the changes to the resource,PATCH is neither safe nor idempotent
|DELETE|DELETE|It is used to **delete** a resource identified by a URI.


## Addressing
> Addressing refers to locating a resource or multiple resources lying on the server. 
* Each resource in REST architecture is identified by its URI (Uniform Resource Identifier).
* A URI is of the following format 
```
<protocol>://<service-name>/<ResourceType>/<ResourceID>

http://localhost:8080/UserManagement/rest/UserService/users/1
```
---

# Building REST services with Spring

## 1. Dao Layer
```
@Entity
class Employee {

  private @Id @GeneratedValue Long id;
  private String name;
  private String role;

  Employee() {}

  Employee(String name, String role) {

    this.name = name;
    this.role = role;
  }

  public Long getId() {
    return this.id;
  }

  public String getName() {
    return this.name;
  }

  public String getRole() {
    return this.role;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public void setName(String name) {
    this.name = name;
  }

  public void setRole(String role) {
    this.role = role;
  }

  @Override
  public boolean equals(Object o) {

    if (this == o)
      return true;
    if (!(o instanceof Employee))
      return false;
    Employee employee = (Employee) o;
    return Objects.equals(this.id, employee.id) && Objects.equals(this.name, employee.name)
        && Objects.equals(this.role, employee.role);
  }

  @Override
  public int hashCode() {
    return Objects.hash(this.id, this.name, this.role);
  }

  @Override
  public String toString() {
    return "Employee{" + "id=" + this.id + ", name='" + this.name + '\'' + ", role='" + this.role + '\'' + '}';
  }
}
```
## 2. Repository
```
package payroll;

import org.springframework.data.jpa.repository.JpaRepository;

interface EmployeeRepository extends JpaRepository<Employee, Long> {

}
```

## 3. Application class
```
package payroll;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class PayrollApplication {

  public static void main(String... args) {
    SpringApplication.run(PayrollApplication.class, args);
  }
}
```

```
package payroll;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
class LoadDatabase {

  private static final Logger log = LoggerFactory.getLogger(LoadDatabase.class);

  @Bean
  CommandLineRunner initDatabase(EmployeeRepository repository) {

    return args -> {
      log.info("Preloading " + repository.save(new Employee("Bilbo Baggins", "burglar")));
      log.info("Preloading " + repository.save(new Employee("Frodo Baggins", "thief")));
    };
  }
}
```
## 4. Controller Layer
```
@RestController
class EmployeeController {

  private final EmployeeRepository repository;

  EmployeeController(EmployeeRepository repository) {
    this.repository = repository;
  }

  // Aggregate root

  @GetMapping("/employees")
  List<Employee> all() {
    return repository.findAll();
  }

  @PostMapping("/employees")
  Employee newEmployee(@RequestBody Employee newEmployee) {
    return repository.save(newEmployee);
  }

  // Single item

  @GetMapping("/employees/{id}")
  Employee one(@PathVariable Long id) {

    return repository.findById(id)
      .orElseThrow(() -> new EmployeeNotFoundException(id));
  }

  @PutMapping("/employees/{id}")
  Employee replaceEmployee(@RequestBody Employee newEmployee, @PathVariable Long id) {

    return repository.findById(id)
      .map(employee -> {
        employee.setName(newEmployee.getName());
        employee.setRole(newEmployee.getRole());
        return repository.save(employee);
      })
      .orElseGet(() -> {
        newEmployee.setId(id);
        return repository.save(newEmployee);
      });
  }

  @DeleteMapping("/employees/{id}")
  void deleteEmployee(@PathVariable Long id) {
    repository.deleteById(id);
  }
}
```

* @RestController indicates that the data returned by each method will be written straight into the response body instead of rendering a template.
* @GetMapping, @PostMapping, @PutMapping and @DeleteMapping, corresponding to HTTP GET, POST, PUT, and DELETE calls)
* @RequestHeader:binds request header values to method parameters
* @ExceptionHandler configures the advice to only respond if an EmployeeNotFoundException is thrown.
* @RequestBody annotation maps the HttpRequest body to a transfer or domain object,
