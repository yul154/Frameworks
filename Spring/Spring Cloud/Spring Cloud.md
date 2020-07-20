# MicroServices
* Autonomously developed
* independently deployed
## Micro
* Each microservice is supposed to do one thing
* Scope of service functionality
* Bounded context: A Microservice is a service built around a specific business capability, which can independently delpoyed
* identify sub-domains: Developers have to identify the sub-domains of the main business domains, and build each-domain as a microservice
## Service
* Interoperability throught message based communication
* Service-oriented architecture(SOA)
* indepedndently deployable component

## Elements
* duplicate the entites that depend on each ther
* sharing databases is discourged and is an empty pattern in the microservice world.
### Data store
* Changes to on service one database doesn't impact any other services
#### Data Synchronization
* no distributed transactions
* no Immediately consistent
* Eventual consistency model: Service publishes an event whe its data changes, the othe service consume event and update that data
* publishing events: Akka, Kafka,RabbitMQ
* Captured data change: Debezium
### Communication
* Describe microservices API using WSDL, Swagger,IDL
#### 1. Remote Procedure InvocationI(RPC):
* interprocess-communication protocol
* Request/reply principle
* Synchronous or Asynchronous
#### Messaging
* microservice exchange messages or events via a broken or channel
* Asynchronous message
* Kafka
* Rabbit MQ
#### Format of data of  exhanges
1. text:XML,JSON,YAML
2. binary: gRPC

### Distrbuted architecture
* by Using a service registry, the client of a microservice will discover its location
* Famous service registries: Eureka,Zookeeper,Consul
* To allow cross-origin resource sharing,microservices have to use additional HTTP header to let a user agent gain permission
#### Service avaiable
* Network failure, Heavy load, DComino effect
* Circuit breaker: invoke a remote service via proxy in order to deviate the call if needed
* If the number of  consecutive failures exceeds a configured limit, a “circuit breaker” will stop attempting to invork . (If a large number of requests are failing, that suggests the service is unavailable and that sending requests is pointless.) After a timeout period, the client should try again and, if the new requests are successful, close the circuit breaker.
* Circuit breaker: Hystrix,JRugged
* Gateway: provide a unified entry point for external consumers, independent of the number and composition of internal microservices.Centrialized user interface calls and access token
  * cross-cutting concerns: authentication, authorization.
  * Zuul, Netty, Finagle
  
### Security
* single sign-on
* Oauth 2.0
* Identity and access management systems: Okta,Keycloak, Shiro.
* Access tokens: Once authenticate, access tokens securely store user information and then exchanged betweens services, token followed JSON Web Token

### Scalability
* Vertical scaling: adding more power to an existing machine
* Horizontal scaling: add more machine
* Client Load Balancing: load balancer will pick up based on criteria
* load-balancing tools: Ribbon,meraki

### Availability
> Probability for a system to be operatinoal at give time.
* Available systems report availability in term of minutes or hours.
* to fix SPOF, all single points o failure need to be scaled horizontally,we have multiple instances which need clustered, so they keep in sync.

### Monitoring
> Centralized monitoring
* Monitoring tools need a dashboard such as kibana, grafana, splunk.to visualize what's wrong.
* Health Check: sometimes a microservice instance can be run but can't handling requests.health check API on each microservice
* Understand th behavior of the application: Each microservice writes all information to a log file which contains errors, warnings, informations
* Log aggregator: don't need to read each log of each microserive,
  * LogStash,Splunk,PaperTrail
  * Exception appear in the middle of log file,so exception should be recorded in a centralized exception.
* Metrics: performance issues,gather statis about each operations
  * Actuator, DropWizard
 * Rate limiting: Control API usage,Dos attacks, limit traffic
 * Auditing:Behavior of users
 * Alerting: Trigger alerts

### Deployment
* Container: one way to simplify package each microservice as container image
* Orchestrator(Kubernetes, Docker swam): running Multiple container across multiple machines.
* Automate deployment: realiable, quick,built, test and deploy
