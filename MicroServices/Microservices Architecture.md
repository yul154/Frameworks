# Introduction of Micro services

## Services
* a pieces of software, which provide functionality to other pieces of software within your system.

### SOA(service oriented architecture)
* scailibility: it allows us to scale up software when demand increases
* resuablility:service structure is planned according to the DRY principle
* standatdized contracts: services follow a standardized description
* The services are stateless: services don’t persist state from former requests
* Service autonomy: services internally control their own logic

# Microservices
* Basically provide services which are more efficiently scalable, flexible
* each one of these microservices provide a set of functions, a set of related functions to specific part o th application.
* Each microservce with a single focus
* lightweight communication mechanism between both client to service an service to serive.
  * transaction which customers carried out is distributed transaction. which complete by multiple services
* Technology agnostic API 
  * service needs to use an open communication protocol(HTTP rest) So that it doesn't dicate the technology that the client application need to uses
* Each microservice has its own data storage 
* Independently changeable:
* Independetly deployable:upgrade service without upgrading any othe part of system.
* Each micro service have its own security mechanism

## Monolithic
* A large service with all the modules packaged together as one executable
* No restriction on size
* One Large codebase: * Have central database in order to share data between applications and services
  * take team longer to develop new functionality with the application
* Fixed techology stack: Can't easily adopt new technologies
* High levesl of coupling: if i change
  * Longer development time: deployment can also be challenging
  * It's difficult to make a change without affecting other parts of system.
  * the code might be intertwined.
* Failure could affect whole system.
* inefficient to scale up
* more powerful resouces cost: in order to run entire application
* Minor change would need longer complie time. More unit test to run
* Pro: run entire code base on one machine, easy to replicate entire environment(configure).


## why micro service
* need to respond to change quickly
* need for reliability: if one part breaks, it won' break the entire system
* Business domain-driven design: 
* Automate test tools: Integration between those services need to be tested
* Release and deployment tools: 
* On-demand hosting technology: 
* Need to adopt new technology
* Asynchronous communication technology: transaction does not have to wait for indivdual services
* Simpler server side and client side
* Shorter development times: different teams are working on different parts concurrently
* Fast issue resolution
* Highly scalable and better performance：just scale specific part up instead of scaling the whole system

## Micro Services Design 
### 1. High Cohesion: 
> Single focus(SOLID prnciple) which can control the size of microserives
* Reason represents: a business function ,a business domain
* Encapsulation principle: OOP
* Easyily rewritable
* Make application high Scalability and flexibility (Change,upgrade the specific business function)
### 2. Autonomous
> Independently changeable, Independetly deployable
* Loose coupling between the microservices: a Change to a microservice should not force others to change
* Stateless: no need to remembe previous interactions
* Backward compatiable
#### Approachs
1. Loosely coupled
* Communication
  1. Synchronous Communication: Making request
  2.Asynchronous: Public event which handled by a message broker,microservice pick up message from a message queue
* Technology agnostic API:Open communication protocols
* Avoid client libraries
* Contract between services: fixed and agreed interface,share model
* Avoid chatty exchanges between services
* Avoid sharing between services: databases
2. Versioning

### 3. Business Domain Centric
> represent business function
* Scope the service and control the size
* Bounded context from DDD(Domain-driven-design)
* approachs: review sub groups of areas, review benefits of splitting further, splitting just for data

### 4. Resilience
> Embrace failure:
* Degrade functionality
* Default functionality
* Multipl instances
* types of failure
* Network issues
#### Approachs
* design for known failures: most of is failure of downstream systems
* one micro service down: degrade the functionality, default functionality
* design system to fail fast: timeout(connect systems)
* Network outages and latency: Monitor timeout, log timeout
 
### 5. Observable
> System Health: 
* Status, Logs, Errors
#### approach
* Centralized MonitoringL Monitor the host, services, Bussiness data related metrics, collect and aggregate monitoring data
* Centralized Logging:record information about events,When to log, Structured logging, traceble distribute transactions

### 6. Automation
> automation tools for testing,feedback for changes and deployment
* reduce test time
* Provide quick feedback
* Provide quick deployment
#### Approach
* Continuouis Integrations tools
  * Work with source control systems
  * Automatic after check-in
  * Unit test and integration tests required
  * Urgency to fix quickly
* Continuouis Deployment tools
  * Configure easily
---------------------------
# Microserice Technology 
## Sychronous Communication
> Make a request and then wait for responses.
* Remote procedure call: Sensitice to change
* HTTP(protocol): Firewall friendlly
 * REST: CRUD using HTTP verbs
* Issues
 * Both parties have to be avaiable
 * Performance subject to network quality
## Asychronous
> event based communication
* When a service needs another service to carry out a task, 
 * instead of connection directly, 
 * the service create an event that services can carry that task out will automatically pick event up
* decouple client and service: There is no need for the client and service to connect 
* Message queueing protocol: Microsoft message queuing(MSMQ), RabbitMQ, ATOM
  * event are seen as messages 
  * publisher: create event service
  * subscriber: Services which carries out tasks in response to the messages is seen as a 
  * message broker: messages are normally stored and buffered until a subscriber pick these messages up.
  * publisher and subscriber only know message broker, so they don't need to know each location
  * easily spawn new versions of microservices and new instances of microservices
  * multiple subscirber can acting on same message.
* weakness: complicated, reliance on message broker
* real-world systems will use both asyn and syn communication

## Hosting platforms

### Vitualization
> one way of hosting micro service.
* A virtual machine as a host that means one physical machine can have multiple virtual machines
* Could be more efficient
* Standardised and mature

### Container
> Another type of vitualization
* Don't running entire operating system within the container
* A good way to isolate services from each other
* Single service per container
* Cloud platform support growing
* Mainly linux based

### Self-hosting
* Implement own cloud
* Use of physical machines

### Registry and Discovery
* Service registry database: when startup, microservices register themself on this database
* Register on startup
* Deregister service on failure
* Cloud platforms make it easy
* Local platform registration options
 1. self registration
 2. Third-party registration
* Local platform discovery options
 1. client-side discovery
 2. Server-side discovery
 
## Observable micro services

### Monitoring tech
* Centralised tools: Nagios, PRTG, Load balancers, New Relic
* Desired features: metrcis acrros servers, automatic or minimal configuration, test transcations support.
* Network monitoring
* Standardise monitoring
* real-time monitoring.
### Logging tech
* Portal for centralised logging data
 * Elastic log
 * Log stash
 * Splunk
 * Kibana
 * Graphite
* store the logging data in a central database
* When a specific event happens within our system. send the log to the cerntralized tools, the tools will store all data as a long entry into a database.
* Client logging libraries: serlog can be configured to push and raise events
* Desired features
 * Structured logging
 * Logging across servers
 * automatic or minimal configuration
* Standardise logging 

## Perfomance
### Scaling
* Automated or on-demand
 * in creating multiple instances of services
 * Adding resource to existing service
* Load balancers

### Caching
* detecting that multiple calls are asking for the same ting 
* honor one request, retrieve the data and then the use the same data to satisfy all others
* Reduce client,service to services and services to databases calls
* API gateway level or proxy level are place to implement ca ching
* Caching can done at client-side level

### API Gateway
* Basically the central entry point into our system for client applications
* Help with performane
 * Load balancing
 * Caching
* Help with
 * creating central entry point
 * Exposing service to clients
 * one interface to many services
 * Dynamic locations of services
 * Routing to specific instance of service
* Security
  * Central security vs service level.
  
  
## Automation tools
## CI/CD
* tools should cross platform
* Source control integration
* notifications
* feedback just received
* test and deploy quicker

# Moving forward with micro services

## Brownfield
> Migration
* Analyst the code in system
* Identify seams: separation that reflects domains
* Identify bounded contexts
### Migration
* Code related to business domain
* Clear boundaries with clear interfaces between each

### Database
* refactor database into multiple database
* Data referential integrity: 
* Static data tables place within a configuration file, make the file avaiable to each microservices. or create an new service.

## Greenfield
> from sratch
* Evolving requirements
* Business domain
 
 
 

