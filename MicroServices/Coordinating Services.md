# Spring Cloud projects features
1. Spring Cloud Eureka: service discovery and service registration,
2. Spring Cloud Hystrix: circuit breakers, this is making sure that when I call those down-stream servers, if it doesn't return a result or has an error, I can handle that really cleanly and then this infrastructure can also keep an eye on that service and when it becomes available again, close the circuit and allow traffic again to go through
3. Spring Cloud Ribbon :  client-side load balancing.
4. Spring Cloud Zuul: an API gateway of sorts that you get to use for routing and all sorts of interesting logic here for sending traffic to the microservices and fronting them with something. It gives you some cool flexibility when you're doing versioning changes or you're addressing different client types. 
5. Spring Cloud Stream: a cool way to abstract out some of the core messaging engines, things like Kafka, RabbitMQ and others so that your code can communicate easily to the messaging backend without having to know a lot about it 
6. Spring Cloud Data Flow: how start to build a data processing pipeline by connecting a set of Spring Boot apps, 

# Service registration and discovery
## The role of service discovery
* Recognize the dynamic environment
* have a live view of healthy services

## Spring Eureka
* REST-based service
* used by Netflix in their AWS Cloud for finding services, for load balancing fail over, and mainly for middle-tier services.

### Dicovering a service with Eureka
* client works with local cache

### Configuring Service Health Information


# Circuit breakers
* When everything is normal, the circuit breaker remains in the closed state and all calls pass through to the services.
* When the number of failures exceeds a predetermined threshold the breaker trips, and circuit  goes into the Open state.
* If the circuit is open, immediately return with an error or a default response
* After a timeout period, the circuit switches to a half-open state to test if the underlying problem still exists
* If a single call fails in this half-open state, the breaker is once again tripped. If it succeeds, the circuit breaker resets back to the normal, closed state. 

## Spring Cloud Hystrix

Netflix has created a library called Hystrix that implements the circuit breaker pattern. 
  * annotation based,Circuit breaker via annotations at class,Hystrix command via annotation at the method
  * Hystrix doesn't just detect failure but helps do something for maintain microservices. Fail fast and rapid recovery.Realtime monitoring and configuration changes. Watch service and property changes take effect immediately as they spread across a fleet.

Thread and semaphore isolation with circuit breakers. 
  * THREAD — it executes on a separate thread and concurrent requests are limited by the number of threads in the thread-pool
  * SEMAPHORE — it executes on the calling thread and concurrent requests are limited by the semaphore coun

Hystrix Dashboard integrates with Eureka to look up services, 
* the set of metrics it gathers about each HystrixCommand. The Hystrix Dashboard displays the health of each circuit breaker in an efficient manner.

### Creating a Hystrix-protected service
Add `spring-cloud-starter-hystrix` 

Annotate class with `@EnableCircuitBreaiker` anntation

Set up `@HystrixXCommand` and define fallback method
*  can use the commandProperties attribute with a list of` @HystrixProperty `annotations.
```
@HystrixCommand(commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "500")
    },
            threadPoolProperties = {
                    @HystrixProperty(name = "coreSize", value = "30"),
                    @HystrixProperty(name = "maxQueueSize", value = "101"),
                    @HystrixProperty(name = "keepAliveTimeMinutes", value = "2"),
                    @HystrixProperty(name = "queueSizeRejectionThreshold", value = "15"),
                    @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "12"),
                    @HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds", value = "1440")
    })
public User getUserById(String id) {
    return userResource.getUserById(id);
    
```
* @DefaultProperties is class (type) level annotation that allows to default commands properties such as groupKey, threadPoolKey, commandProperties, threadPoolProperties, ignoreExceptions and raiseHystrixExceptions. 

### Hystrix Dashboard
functions.
  * can see circuit breaker status
  * error percentage
  * request rate
  * latency percentiles
  *  Number of hosts in the cluster
  
Create
* create new project
* Choose Hystrix Dashboard. dependency
* Annotate `@EnableHystrixDashboard` on main class(eruka register=false)
* set up in application.properity
* Turbine combine dividual instance into one overall view


# Client-side load balancing and API Proxying requests between services

Role of Routing
* rapid decision making
* cross-cutting concerns(authentication, caching,logging, rate limiting)
* offer aggregation to clients 

## Spring Cloud Ribbon
> Client-side load balancer which can do round-robin load balancing
* storage of server addresses (server list), server freshness checks (ping), are these servers online and available. server selection criteria(Rules)

Annotation 
* @LoadBalanced : indicating that the annotated RestTemplate should use a RibbonLoadBalancerClient for interacting with your service(s)
* @RibbonClient:Used for configuring your Ribbon client if  not using any service discovery

Configure

<img width="599" alt="Screen Shot 2019-10-05 at 12 54 00 PM" src="https://user-images.githubusercontent.com/27160394/66258102-3a7a7200-e76f-11e9-94fc-d1a5bf79e3e0.png">

Without Service Discovery
* the name field of the @RibbonClient annotation will be used to prefix your configuration in the application.properties 
* then in your application.properties, add services list.

With Eureka
* Server list comes from Eureka server
* Ribbon cache comes from Eureka client local


Customize Configure
* Override behavior in @Configuratio class
* or, Also do this in propoerties

## Spring zuul
> Embeded  proxy fo routing traffic

* a unique gateway to the client applications of your system

Configuration

```
server.port=8091
spring.application.name=06-springcloud-api-gateway

zuul.ignoredServices=*
zuul.routes.api-kinglong.path=/api-kinglong/**  //fix end
zuul.routes.api-kinglong.service-id=05-springcloud-service-feign
zuul.prefix=/api
eureka.client.service-url.defaultZone=http://eureka8085:8085/eureka/,http://eureka8086:8086/eureka/

```
### Filters and Stages in Zuul

<img width="477" alt="Screen Shot 2019-10-05 at 2 20 04 PM" src="https://user-images.githubusercontent.com/27160394/66259627-1f186280-e781-11e9-8396-fe80d5c39b3a.png">

Filters
* pre filters are executed before the request is routed,
* route filters can handle the actual routing of the request,
* post filters are executed after the request has been routed,
* error filters execute if an error occurs in the course of handling the request.
* wide range of standard filters

```
public class SimpleFilter extends ZuulFilter {

  private static Logger log = LoggerFactory.getLogger(SimpleFilter.class);

  @Override
  public String filterType() {
    return "pre";
  }

  @Override
  public int filterOrder() {
    return 1;
  }

  @Override
  public boolean shouldFilter() {
    return true;
  }

  @Override
  public Object run() {
    RequestContext ctx = RequestContext.getCurrentContext();
    HttpServletRequest request = ctx.getRequest();

    log.info(String.format("%s request to %s", request.getMethod(), request.getRequestURL().toString()));

    return null;
  }

}
```
* Create bean in main class

# Connecting Microservices Through Messaging

Role of Messaging

* messaging helps make that Microservices evolve independently
* New intake and processong patterns

## Spring Cloud Stream
> Framework for building message-driven microservice apps

MessageChannel an Message<T>
 * Message object has a payload and a set of headers to provide some metadata about the payload itself. 
 * MessageChannel's a lot like a Java Queue. 

Channel Adapters
* takes a message and then sends that to some downstream system.
* adapters make it simple to connect to these messaging sources

ServiceActivator
* the trigger activate a spring-managed service object.
* This activator can pull the MessageChannel looking for messages
*  accept a parameter of type Message or of the expected Message payload's type

## Core conecpts in Spring Cloud Stream

Channel
 * decoupling some of those components in messaging engines with that MessageChannel object
 * A channel represents an input and output pipe between the Spring Cloud Stream Application and the Middleware Platform.


Bindings — a collection of interfaces that identify the input and output channels declaratively。
* Binder —  connecting to physical destinations at the external middleware
* It defines this logical reference to other services, 
* so it helps me connect to s messaging channel but you don't have to know the API. It's a RabbitMQ or Kafka or Google Pub/Sub.

@Streamlistener
>  a listener to inputs declared via EnableBinding (e.g. channels). 

* Handler for inbound messages
* automatic content type conversion

Pub/Sub Pattern
> Publishers categorize messages into topics, each identified by a name. Subscribers express interest in one or more topics. The middleware filters the messages, delivering those of the interesting topics to the subscribers.
* All the bindings are pub/sub by default. 
* You can make things appear to be more one to one and do some things 
* this reduces that complexity of adding any new apps without disrupting the flow. 
* the subscribers could be grouped. A consumer group is a set of subscribers or consumers, identified by a group id, within which messages from a topic or topic's partition are delivered in a load-balanced manner.


Consumer groups
* If I want to have multiple instances of a receiving app and I'll put them in this sort of competing consumer relationship where only one of those instances will actually pull it. 

Partition
* everything feel stateless

<img width="526" alt="Screen Shot 2019-10-05 at 11 04 10 PM" src="https://user-images.githubusercontent.com/27160394/66263658-8277b400-e7c4-11e9-9a10-3f82a6ee18a4.png">



## Binder and Configuriong Spring Stream

Binder
* Spring Cloud detect binders on classpath
* can connect to multipl brokers of sam type
* Different binder with same code

Configuration

###  binding interfaces for typical message exchange contracts
 * Sink: Identifies the contract for the message consumer by providing the destination from which the message is consumed.
 * Source: Identifies the contract for the message producer by providing the destination to which the produced message is sent.
 * Processor: Encapsulates both the sink and the source contracts by exposing two destinations that allow consumption and production of messages.
 
Sender

```
@EnableBinding(Source.class) // the class as spring stream application
public class TimerSource {

  @Bean
  // Indicates that a method is capable of producing a Message or Message payload by using return value. sends a message to the source’s output channel
  @InboundChannelAdapter(value = Source.OUTPUT, poller = @Poller(fixedDelay = "10", maxMessagesPerPoll = "1")) 
  public MessageSource<String> timerMessageSource() {
    return () -> new GenericMessage<>("Hello Spring Cloud Stream");
  }
}
```
```
spring.cloud.stream.bindings.output.destination=orders create/connect queue call orders

spring.rabbitmq.host=
spring.rabbitmq.port=
spring.rabbitmq.username=
spring.rabbitmq.password=
```

Receiver
```
@SpringBootApplication
@EnableBinding(Sink.class)
public class VoteRecordingSinkApplication {

  public static void main(String[] args) {
    SpringApplication.run(StreamReceiver.class, args);
  }

  @StreamListener(Sink.INPUT)
  public void log(String msg) {
      System.out.println(msg);
  }
}
```
```
spring.cloud.stream.bindings.inputt.destination=orders create/connect queue call orders

spring.rabbitmq.host=
spring.rabbitmq.port=
spring.rabbitmq.username=
spring.rabbitmq.password=
```
* More options
 1. Customize behavior of InboundChannel Adapter(batching)
 2. Create custom interfaces for channel definitions
 3. Set "condition" on @StreamListener to dispatch to different methods whih can filter the messages we expect in the consumer

Processor
* sends and receives
* have inbound and outbound channel
* Use @SendTo to set output destination for the data returned by a method
* Possible to use different broker instances or type for channels
 
### Customize interface for sender application
```
package pluralsight.demo;

import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;

public interface TollSource {
	
	@Output("fastpassTollChannel")
	MessageChannel fastpassToll();
	
	@Output("standardTollChannel")
	MessageChannel standardToll();
}
```

```

@EnableBinding(someSource.class)
public class TollPublisher {

	//@InboundChannelAdapter(channel=Source.OUTPUT)
	//public String sendTollCharge() {
	//	return "{station:\"20\", customerid:\"100\", timestamp:\"2017-07-12T03:15:00\"}";
	//}
	
	Random r = new Random();
	
	@Bean
	//@InboundChannelAdapter(channel="fastpassTollChannel", poller=@Poller(fixedDelay="2000"))
	public MessageSource<Toll> sendTollCharge() {

		return () -> MessageBuilder.withPayload(new Toll("20","100","2017-07-12T12:04:00")).setHeader("speed", r.nextInt(8) * 10).build();		
	}
	
	class Toll {
		public String stationId;
		public String customerId;
		public String timestamp;
		
		public Toll(String StationId, String CustomerId, String Timestamp) {
			this.stationId = StationId;
			this.customerId = CustomerId;
			this.timestamp = Timestamp;
		}
	}
	
}
```

## Consumer Groups
> a set of subscribers or consumers, 
Use to scale up subscribers, Message goes to single instance in each group

*  durable subscrption : if stop consumer, then the subscription goes away. Messages that keep coming in would never be picked up by the consumer.
*  used to instances all point to the exact same queue and have this competing consumer model

* Set spring.cloud.stream.bindings.<channelName>.group
 
 ## Partition
 * Modeled after Kafka 
 * Data split, processed by unique consumer instance
 
 ## contentType
 * contentType headed on outbound messages
 * Use bean for custom message converters
 
