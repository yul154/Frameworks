# ? Different betwee monolith and micro service
* Microservices, aka Microservice Architecture, is an architectural style that structures an application as a collection of small autonomous services, modeled around a business domain.
* Monolith: all the software components of an application are assembled together and tightly packaged.
## Pro of micro service
* Independent Development: Each microservce with a single focus,Easier to write and Maintain code
* Independent deloyment: As a result, it makes continuous deployment possible for complex applications.
* Mixed Technology Stack: Different languages and technologies can be used to build different services of the same application
* scale: Individual components can scale as per need, you can scale them separately, whenever it’s necessary. there is no need to scale all components together
* Fault Isolation: isolated services have a better failure tolerance, if one part breaks, it won' break the entire system

## Cons of micro-services
* Developing distributed systems can be complex
  * Since everything is now an independent service, you have to carefully handle requests traveling between your modules
* choose and implement an inter-process communication mechanism based on either messaging or RPC and write code to handle partial failure 
* Security issues (harder to maintain transaction safety, distributed communication goes wrong more likely
* Deploying a microservices-based application is also more complex. A monolithic application is simply deployed on a set of identical servers behind
  * In contrast, a microservice application typically consists of a large number of services. Each service will have multiple runtime instances. And each instance need to be configured, deployed, scaled, and monitored. 
* Testing a microservices-based application can be cumbersome. 
  *  In a monolithic approach, we would just need to launch our WAR on an application server and ensure its connectivity with the underlying database. 
  *  With microservices, each dependent service needs to be confirmed before testing can occur
---------  
# How micro service communicate in your project

* Synchronous one to one : request/response‑based IPC mechanism, a client sends a request to a service. The service processes the request and sends back a response.
* ASynchrnous one to one :, one producer will send the messages to a message broker and one consumer receive the message and process it accordingly.
* one to many(Pub/Sub Pattern): Publishers categorize messages into topics, each identified by a name. Subscribers express interest in one or more topics. The middleware filters the messages, delivering those of the interesting topics to the subscribers.the subscribers could be grouped. A consumer group is a set of subscribers or consumers, identified by a group id, within which messages from a topic or topic's partition are delivered in a load-balanced manner.
  
  
# what is  fault tolerance, how to do fault tolerance in your project, give me a real-time example

* Circuit breaker pattern: the caller not to attempt to make a call to the dependent service, which is likely to fail. And instead, the caller should fail fast or degrade gracefully, perhaps by returning old data or empty results, and allow the failing service to recover. And then periodically check if the service has recovered.

* fallback: Any requests that are short-circuited or timed-out or rejected or failed will execute
returning cache data, a default value or empty response

* Netflix Hystrix implemented Circuit breaker pattern

* security
  *  API Gateway Pattern: single sign-on, It can also help enable observability metrics 
 
## Handling Failures
 * The Circuit Breaker(hystrix)
   * if a downstream service is down for a certain time and trips the circuit to avoid sending calls to it. It retries to check again after a defined period if the service has come back up and closes the circuit to continue the calls to it.
 * Bulkhead pattern: isolate the resources used for a service and avoid cascading failures
 * Spring Cloud Hystrix applies both Circuit Breaker and Bulkhead patterns.
 * Provide fallbacks. In this approach, the client process performs fallback logic when a request fails, such as returning cached data or a default value.
 * Limit the number of queued requests: Clients should also impose an upper bound on the number of outstanding requests that a client microservice can send to a particular service.
 * set network connect and read timeouts to avoid waiting too long for the response. 
 
# WHAT IS API GATE
* An API Gateway: generally used for managed APIs where it handles requests from UIs or other consumers and makes downstream calls to multiple microservices and responds back.

# Define the architecture  Netflix and os

Service Discovery Server(Netflix Eureka ) is the directory of the system allowing everyone to register their location, as well as discover the location of others.

the Config Server(Spring Cloud Config Server): to handle our dynamic and distributed configuration needs.

the gateway(zuul) : responsible for receiving and routing requests to back-end services. use Netflix Zuul

client-side load balancer(Ribbon): distribute requests among the multiple instances that we run for high availability.

fault tolerance(Netflix Hystrix): tolerate and measure failures and prevent them from causing cascading failures to other systems.

# IF rest API not repsonding , what respond you will create to Get restart ,what response you will create

* Use the Circuit Breaker pattern. In this approach, the client process tracks the number of failed requests. If the error rate exceeds a configured limit, a “circuit breaker” trips so that further attempts fail immediately. (If a large number of requests are failing, that suggests the service is unavailable and that sending requests is pointless.) After a timeout period, the client should try again and, if the new requests are successful, close the circuit breake
* fallback function
* Provide fallbacks. In this approach, the client process performs fallback logic when a request fails, such as returning cached data or a default value. This is an approach suitable for queries, and is more complex for updates or commands.




