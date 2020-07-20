# Finding services using Dicovery
> How do you dynamically find your services in Cloud at runtime

## How does one service know where anothe service is at?
### Service discovery provide
  1. a way for a service to register(deregister) itself
  2. a way for a client to find other services
  3. Check the health of a service and remove unhealthy instances
  
### Key Components in Service Discovery
#### 1. Discovery server: 
* a service registry (Eureka Server
* An actively managed registry of service locations
* Eureka Server
  * configuration
```
# pom.xml
<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Camden.SR2</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
</dependencyManagement>
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

# application.properties
spring.application.name=discovery-server
eureka.client.registerWithEureka = false#  registers itself into the discovery
eureka.client.fetchRegistry = false
server.port = 8761

# In main application class
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class EurekaServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServiceApplication.class, args);
    }
}
````
#### 2. Application service
* a REST service which registers itself at the registry (Eureka Client)
* Provide some application functionality
* The receiver of requests
* One or more instances
* User of discovety client(register)
```
# POM.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka</artifactId>
    <version>2.1.3.RELEASE</version>
</dependency>

# application.properties
spring.application.name=service
eureka.client.service-url.default.Zone=http://localhost:8761/eureka // where the service discovery servce located

# In main application class
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServiceApplication.class, args);
    }
}

```
#### 3. Client
* Call anothe application service to implement its functionality
* User of discovety client(find service locations)
* The different betwee applicatio service and application client in configuration
```
# application.properites
spring.application.name=client
eureka.client.service-url.default.Zone=http://localhost:8761/eureka // where the service discovery servce located
eureka.client.register-with-eureka=false

# In main application class
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServiceApplication.class, args);
    }
}
```
* Discovering services as clients
  1. Inject EurekaClient
  2. Inject DiscoveryClient

### Eureka DashBoard
 * http://localhost:8761  to see its registration status on the Eureka Dashboard. 

### Eureka Configuration
1. eureka.server.*(discover server)
2. eureka.client.*(how the discovery client communicate interact wit the discovery server)
3. eureka.instance.*

### Health& high availability
* regular check the status of services
* sending heartbeats every 30 sec
* The registry is distributed

### AWS support
* Configuring Eureka for AWS
```
@Bean
public EurekaInstanceConfigBean eurekaInstanceConfig() {
    EurekaInstanceConfigBean bean = new EurekaInstanceConfigBean();
    AmazonInfo info = AmazonInfo.Builder.newBuilder().autoBuild("eureka");
    bean.setDataCenterInfo(info);
    return bean;
  }
```
* `eureka.client.availability-zones.[region]=[az1],[az2]`


# Configuring Services Using Distributed Configuration
## distributed VS non-distributed
* in distributed application, you typically only have a handful of configuration files. 
* Configuration server: dedicated,dynamic, centralized key/value store.

## Managing Application Configuration with Spring Cloud
* SC provide several ways to implement a Configuration server
  * Spring Cloud Consul 
  * Spring Cloud Zookeeper 
  * Spring Cloud Config(one focus)
* Config server is 
  * a standalone web application, 
  * integrate with property source.
  * provide a rest-based interface for access
  * Outputformate: JSON(default),properties, YAML
  * no database to store, use git(default),SVN
  * Configuration scopes: spring profile configuration, global configuration and application-specific configuration
### Configure config server
```
// POM.XML

<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>{spring-cloud-version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
* Create a folder to store configuration
  * optional add a propertoes or yml file 
  * add profile-specific configuraton files in this folder with specifl name pattern `{application}-{profile}`
```
// application.properties
server.port=8888
spring.cloud.config.server.git.uri=<uri_to_git_repo>
```
```
// main class
@EnableConfigServer
@SpringBootApplication
public class ConfigurationServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigurationServiceApplication.class, args);
    }
}
```
### config rest Endpoints
* Common sets of parameters
  1. application: maps to spring.application.name on client
  2. profile: mapt to spring.profiles.active on client,(default is default)
  3. label: group you configuration file into kind of arbitrary name set(git branch)
```
GET/{application}/{profile}/[/{label}]
/myapp/dev/master //myapp is application anme, dev is profile and master is git branch
```
* request properoties file
   * `/myapp-dev.yml`
   * `/myapp-prod.properties`
   * `/myapp-default.[ropperties`.
   
### Config Client
* Config Client needs to fetch the application configuration before the Spring application context has even technically started.
* bootstrap your application properties(`bootstrap.properoties`,`bootstrap.yml`)
   1. Config first: Configuring a bootstrap file that hash the application name to the configuration server
   2. Discovery first:using service discovery, configure bootstrap file(have the application name), and then location of Discovery server. It would use that to then find the Config Server so that it could fetch your configuration.
 
* Setting up an application to use the Spring Cloud Config Client
   * import dependency of spring cloud within depdendency management section
   * Config First
   ```
   spring application.name=<your_app_name>
   spring.cloud.config.uri=http://localhost:8888/ //location of config server
   ```
   * discovery first
   ```
   spring application.name=<your_app_name>
   spring.cloud.config.discovery.enableed=true
   ```
#### Updating Configuration at Runtime
* Refresh(@ConfigurationProperties) in spring-boot-actuator
* if each of the applications were to subscribe to an event,and you were to call the bus/refresh endpoint, Spring Cloud Bus would send out a message to all of the subscribers indicating to them that they need to refresh their configuration
* Update logging levels
* Monitor
* Using `@RefreshScope`in `@Configuration` class to bean: allows for beans to be refreshed dynamically at runtime 


# Intelligent routing
* Each of the individual services may be running on a different port, a different address, or a combination of both.

## Gateway
>  a single entry point for all clients, which kind of front door
*  gateway service(API gatway) provide dynamic routing and delivery
   *  at runtime it can decide where it should route a request and if it should even route a request at all
* provide security, it provides the ability to authenticate all of the incoming requests as well as filter
* Auditing and logging
* request enhancement: add that additional information as an additional request header 
* load balancing
* different APIs for different clients

## Netflix Zuul
* a gateway service that provide dynamic routing, monitoring, resiliency, sercurity and more
* Configuration
  * import spring cloud within pom.xml
  * import starter-zuul dependency
  * add `@EnableZuulProxy` Annotation on main class
  * if using discovery, add eureka configuration i properties
  * if not using discovery, `ribbon.enreka.enabled=false`
  * the name of the client-side load balancer is Ribbon.
* If you were to request /categories/1, Zuul would locate the service with the name categories(spring.application.name), 
    * it would send the /1 request to that service. 
* Configure route in Zuul
  * with service discovery
 ```
 spring.application.name=gateway-service
 zuul routes.<route_name>.path=/somepath/**
 zuul routes.<route_name>.serviceId=some_service_id
 zuul.ignored-services=some_service_id
 ```
   * without service discovery
```
spring.application.name=gateway-service
zuul routes.<route_name>.path=/somepath/**
zuul routes.<route_name>.url=http://some_url_address/
zull routes.<route_name>.serviceId=some_service_id
```
### Filters
* Types
  1. pre:executed before the request is routed
  2. route: after pre filters and make requests to other services,translate request and response data to and from the model required by the client
  3. post
  4. error
 * define zuul filter
 ```
public class QueryParamPreFilter extends ZuulFilter {
	@Override
	public int filterOrder() {
		return PRE_DECORATION_FILTER_ORDER - 1; // run before PreDecoration
	}

	@Override
	public String filterType() {
		return PRE_TYPE;
	}

	@Override
	public boolean shouldFilter() {
		RequestContext ctx = RequestContext.getCurrentContext();// hold request,response, state and data
		return !ctx.containsKey(FORWARD_TO_KEY) // a filter has already forwarded
				&& !ctx.containsKey(SERVICE_ID_KEY); // a filter has already determined serviceId
	}
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
		HttpServletRequest request = ctx.getRequest();
		if (request.getParameter("sample") != null) {
		    // put the serviceId in `RequestContext`
    		ctx.put(SERVICE_ID_KEY, request.getParameter("foo"));
    	}
        return null;
    }
}
```
 * define an `@Bean` Which return the filter in @Configuration class
 
# Loading Balancing
 * ways to improve the distribution workloads across multiple computing resources
 
## Types of load balancing
1. server-side load balancer
  * a request to another service doesn't go directly to the service itself and instead goes to a server in front of the service, which then decides which of the multiple instances it should forward the request to. 
 <img width="257" alt="Screen Shot 2019-10-04 at 1 54 59 PM" src="https://user-images.githubusercontent.com/27160394/66228916-a77a0300-e6ae-11e9-81c1-a1dc223284a3.png">

2.  client-side load balancer
  * the client itself, is responsible for deciding where to send the traffic also using an algorithm like round-robin. It can either discover the instances, via service discovery, or can be configured with a predefined list.
 
 <img width="244" alt="Screen Shot 2019-10-04 at 1 56 27 PM" src="https://user-images.githubusercontent.com/27160394/66228964-c7112b80-e6ae-11e9-8fea-9c090c5fa787.png">

## Client-side load balancing with Spring Cloud
* Netflix Ribbon: an Inter Process Communication (IPC) cloud library. Ribbon primarily provides client-side load balancing algorithms.
* Spring Cloud adds full integration with Netflix Ribbon to Spring's `RestTemplate` Class
* Customize configuration for different(algirthms, availability checks)

###  Configurate Netflix Ribbon
* In `pom.xml`
  * define a new dependency on spring-cloud-dependencies In dependencyManagement section.make sure that it's of type pom and has a scope of import.
  *  define a new dependency on `spring-cloud-starter-ribbon` in the dependency section.
  
####  Annotations
1. `@LoadBalanced`: Marks a `RestTemple` to support load balancing
* Create a Load Balanced RestTemplate with service discovery
* Instead of passing the mycompany. com URL or IP address to the RestTemplate, you can actually pass a URL that uses a logical identifier which is the same name in service discovery server.
 *  at runtime the RestTemplate will function as the client-side load balancer. And it'll use service discovery to resolve the real location of the my-service instances and then use the configured load balancing algorithm to distribute the load between them.


```
@ Configuration
public class MyConfiguration{
    @Bean
    @LoadBalanced
    public RestTemplate restTemplagte(){
    	return new RestTemplate();// will return interceptor
    }
	
}
```

 
2.  client-side load balancing without service discovery: uses `@RibbonClient` along with the `@LoadBalanced `annotation to achieve 
  1. In `@Configuration` class, define the `@RibbonClient` annotation and set the name element to a meaningful value. 
  2. in Application.property
  
<img width="979" alt="Screen Shot 2019-10-04 at 2 43 34 PM" src="https://user-images.githubusercontent.com/27160394/66231809-61746d80-e6b5-11e9-8a77-155137757524.png">

  3 `RestTemplate`: instead of calling the service name, you use the value of the name element that you set up in the @RibbonClient annotation.
  4. delete the property that sets the location of the Service Discovery Server. 
  
* Custom `RibbonClient` configuration
  * custom configuration that applies to a specific Ribbon client instead of to all Ribbon clients
  * in @Configuration class,define @RibbonClient
```
package io.schultz.dustin;
@Configuration
@RibbonClient(
    name = "otherservice",
    configuration = OtherServiceConfig.class)
public class MyConfiguration {
...
}
```
 * different Bean that are required to set up a Ribbon client:
   * `IRule` bean: control load balancing algorithm
   * `IPing`:  choosing the strategy to check the liveliness or the availability of a given instance that's being load balanced. 
				
				
# Fault tolerant and self-healing
## failure
* Failure: hardward fail, network fail and software fail
* Cascading failure: failure of one or few parts can trigger the failure of other parts 
* Fault tolerance problem: Calling services are unaware that the service that they're calling is likely to fail and yet they still attempt to call the service
* resource overloading problem
## Strategy for failure
*  Circuit breaker pattern: the caller not to attempt to make a call to the dependent service, which is likely to fail. And instead, the caller should fail fast or degrade gracefully, perhaps by returning old data or empty results, and allow the failing service to recover. And then periodically check if the service has recovered.
* limit the resources that are consumed： So clients should put limits on the number of resources allowed to call a dependent service.

## Fault tolerance with Netflix Hystrix 
* Implementation of  the circuit breaker pattern
* wraps calls and automatically watcjes them fo failures that meet a certain volumn
* default for rilling window is 10 seconds
* 20 request volume
* >= 50 % request error rate: the circute will be tripped and no requests will allowed throug
* waits and tries a single request fate 5 sec
* fallback: Any requests that are short-circuited or timed-out or rejected or failed will execute 
   * returning cache data, a default value or empty response
* Hystrix also has additional functionality that protects services from being overloaded
  * All Hystrix wrap calls are bounded either by a thread pool or a semaphore. And what this does is it constrains the resource usage, 

## Using netflix hystrix
### Configuration
* pom.xml : spring cloud,starter-hystrix,actuator
```
<dependencyManagement>
   <dependencies>
	<dependency> 
		<groupId>org.springframework.cloud</groupId> 
		<artifactId>spring-cloud-dependencies</artifactId>
		<version>Camden.SR2</version>
		<type>pom</type>
		<scope>import</scope>
       </dependency>
    </dependencies>
</dependencyManagement>

<dependency>
	<groupId>org.springframework.cloud</groupId> <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
<dependency> 
	<groupId>org.springframework.boot</groupId> <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
* Applicaton.java
```
@SpringBootApplication
@EnableCircuitBreaker
public class Application {
   public static void main(String[] args) {
       SpringApplication.run(Application.class, args);
   }
}
```
* Service.java: in eithe your @Component or @Service
```
@Serivce
public class Service {
   @HystrixCommand(fallbackMethod = "somethingElse") 
   public void doSomething() {
		...
   }
   public void somethingElse() {
	...
   }

}
```
# Fit together
* Service Discovery Server(Netflix Eureka ) is the directory of the system allowing everyone to register their location, as well as discover the location of others. 
* the Config Server(Spring Cloud Config Server):  to handle our dynamic and distributed configuration needs. 
* the gateway(zuul) : responsible for receiving and routing requests to back-end services. use Netflix Zuul 
* client-side load balancer(Ribbon): distribute requests among the multiple instances that we run for high availability. 
* fault tolerance(Netflix Hystrix): tolerate and measure failures and prevent them from causing cascading failures to other systems.

### Monitor Hystrix Metrics in real-time with the Hystrix Dashboard
```
<dependency>
<groupId>org.springframework.cloud</groupId> <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>

@SpringBootApplication
@EnableHystrixDashboard
public class Application {
   public static void main(String[] args) {
       SpringApplication.run(Application.class, args);
   }
}
```
* Aggregating Hystrix Streams with Turbine
