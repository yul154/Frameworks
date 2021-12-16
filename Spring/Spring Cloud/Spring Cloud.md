
* 集群：同一个业务，部署在多个服务器上(不同的服务器运行同样的代码，干同一件事)
* 分布式系统是一组计算机，通过网络相互连接传递消息与通信后并协调它们的行为而形成的系统

# Spring Cloud


SpringCloud 在CAP理论是选择了AP的

<img width="550" alt="Screen Shot 2021-12-16 at 6 38 21 PM" src="https://user-images.githubusercontent.com/27160394/146356187-d567f693-6890-4cf3-8b7d-95639a9072c7.png">


```
服务治理：Spring  Cloud Eureka

客户端负载均衡：Spring Cloud Ribbon

服务容错保护：Spring  Cloud Hystrix  

声明式服务调用：Spring  Cloud Feign

API网关服务：Spring Cloud Zuul

分布式配置中心：Spring Cloud Config
```

* Spring Cloud Eureka: service discovery and service registration,
* Spring Cloud Hystrix: circuit breakers, this is making sure that when I call those down-stream servers, if it doesn't return a result or has an error, I can handle that really cleanly and then this infrastructure can also keep an eye on that service and when it becomes available again, close the circuit and allow traffic again to go through
* Spring Cloud Ribbon : client-side load balancing.
* Spring Cloud Zuul: an API gateway of sorts that you get to use for routing and all sorts of interesting logic here for sending traffic to the microservices and fronting them with something. It gives you some cool flexibility when you're doing versioning changes or you're addressing different client types.
* Spring Cloud Stream: a cool way to abstract out some of the core messaging engines, things like Kafka, RabbitMQ and others so that your code can communicate easily to the messaging backend without having to know a lot about it
Spring Cloud Data Flow: how start to build a data processing pipeline by connecting a set of Spring Boot apps,

-----
# Eureka
> Eureka 就是一个服务发现框架


Service discovery provide
* a way for a service to register(deregister) itself
* a way for a client to find other services
* Check the health of a service and remove unhealthy instances

server可以拿到Eureka(服务E)那份注册清单
*  互相调用不再通过具体的IP地址，而是通过服务名来调用


### Key Components in Service Discovery

1. Discovery server(erueka)
 * A service registry (Eureka Server
 * An actively managed registry of service locations 
```
spring.application.name=discovery-server
eureka.client.registerWithEureka = false#  registers itself into the discovery
eureka.client.fetchRegistry = false
```

2. Application service
 * a REST service which registers itself at the registry (Eureka Client)
 * Provide some application functionality
 * The receiver of requests
 * User of discovety client(register) 

```
spring.application.name=service
eureka.client.service-url.default.Zone=http://localhost:8761/eureka // where the service discovery servce located
```

3. Client
 * Call anothe application service to implement its functionality
 * User of discovety client(find service locations)
 * The different betwee application service and application client in configuration

```
spring.application.name=client
eureka.client.service-url.default.Zone=http://localhost:8761/eureka // where the service discovery servce located
eureka.client.register-with-eureka=false
```

### Eureka的治理机制
* 服务提供者
 *  服务注册：启动的时候会通过发送REST请求的方式将自己注册到Eureka Server上
 *  服务续约：在注册完服务之后，服务提供者会维护一个心跳用来持续告诉Eureka Server,当Eureka客户端连续90秒(3个续约周期)没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除
 *  服务下线：当服务实例进行正常的关闭操作时，它会触发一个服务下线的REST请求给Eureka Server,

* 服务消费者
 * 获取服务：当我们启动服务消费者的时候，它会发送一个REST请求给服务注册中心，来获取上面注册的服务清单
 * 服务调用：服务消费者在获取服务清单后，通过服务名可以获得具体提供服务的实例名和该实例的元数据信息。在进行服务调用的时候，优先访问同处一个Zone中的服务提供方
 
* Eureka Server(服务注册中心)：
 * 失效剔除：默认每隔一段时间（默认为60秒） 将当前清单中超时（默认为90秒）没有续约的服务剔除出去
 * 自我保护。EurekaServer 在运行期间，会统计心跳失败的比例在15分钟之内是否低于85%
---
# Ribbon

> RestTemplate是Spring提供的一个访问Http服务的客户端类, 一般在使用SpringCloud的时候不需要自己手动创建HttpClient来进行远程调用。可以使用Spring封装好的RestTemplate工具类


 <img width="257" alt="Screen Shot 2019-10-04 at 1 54 59 PM" src="https://user-images.githubusercontent.com/27160394/66228916-a77a0300-e6ae-11e9-81c1-a1dc223284a3.png">

Server-side load balancer(Nginx)
* a request to another service doesn't go directly to the service itself and instead goes to a server in front of the service, which then decides which of the multiple instances it should forward the request to.
* 将所有请求都集中起来，然后再进行负载均衡


 <img width="244" alt="Screen Shot 2019-10-04 at 1 56 27 PM" src="https://user-images.githubusercontent.com/27160394/66228964-c7112b80-e6ae-11e9-8fea-9c090c5fa787.png">

client-side load balancer(Ribbon)
* The client itself, is responsible for deciding where to send the traffic also using an algorithm like round-robin. It can either discover the instances, via service discovery, or can be configured with a predefined list.
* client端获取到了所有的服务列表之后，在其内部使用负载均衡算法，进行对多个系统的调用



##  Ribbon 的几种负载均衡算法
> 其默认是使用的RoundRobinRule轮询策略。

* RoundRobinRule：轮询策略。Ribbon默认采用的策略。若经过一轮轮询没有找到可用的provider，其最多轮询 10 轮。若最终还没有找到，则返回 null。

* RandomRule: 随机策略，从所有可用的 provider 中随机选择一个。

* RetryRule: 重试策略。先按照 RoundRobinRule 策略获取 provider，若获取失败，则在指定的时限内重试。默认的时限为 500 毫秒。



##  Configurate Netflix Ribbon

Ribbon是支持负载均衡，默认的负载均衡策略是轮询

```
providerName:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
    

@RibbonClient 
@LoadBalanced

@Configuration
public class MySelfRule
{
    @Bean
    public IRule myRule()
    {
        //return new RandomRule();// Ribbon默认是轮询，我自定义为随机
        //return new RoundRobinRule();// Ribbon默认是轮询，我自定义为随机

        return new RandomRule_ZY();// 我自定义为每台机器5次
    }
}
```


----
# Hystrix(Fault tolerance)
> 在高并发的情况下，由于单个服务的延迟，可能导致所有的请求都处于延迟状态，甚至在几秒钟就使服务处于负载饱和的状态，资源耗尽，直到不可用，最终导致这个分布式系统都不可用，这就是雪崩


* 熔断(断路器模式): 服务雪崩的一种有效解决方案。当指定时间窗内的请求失败率达到设定阈值时，系统将通过断路器直接将此请求链路断开
* 降级(fallback) : 为了更好的用户体验，当一个方法调用异常时，通过执行另一种代码逻辑来给用户友好的回复

> 总体来说Hystrix就是一个能进行熔断和降级的库，通过使用它能提高整个系统的弹性。



Spring Cloud Hystrix实现了断路器、线程隔离等一系列服务保护功能

```
 @HystrixCommand(fallbackMethod = "somethingElse") 
```

`Fallback(失败快速返回)`：当某个服务单元发生故障（类似用电器发生短路）之后，通过断路器的故障监控（类似熔断保险丝）, 向调用方返回一个错误响应， 而不是长时间的等待
* 这样就不会使得线程因调用故障服务被长时间占用不释放，避免了故障在分布式系统中的蔓延。

Hystrix提供几个熔断关键参数：滑动窗口大小（20）、 熔断器开关间隔（5s）、错误率（50%）
* 每当20个请求中，有50%失败时，熔断器就会打开，此时再调用此服务，将会直接返回失败，不再调远程服务。
* 直到5s钟之后，重新检测该触发条件，判断是否把熔断器关闭，或者继续打开。


舱壁模式
* 资源/依赖隔离(线程池隔离)：它会为每一个依赖服务创建一个独立的线程池，这样就算某个依赖服务出现延迟过高的情况，也只是对该依赖服务的调用产生影响， 而不会拖慢其他的依赖服务。
* 舱壁模式会将远程资源调用隔离在他们自己的线程池中，以便可以控制单个表现不佳的服务，而不会使该程序崩溃

##  Hystrix Dashboard
> 实时监控Hystrix的各项指标信息


-----
# Feign

* 整合了 Spring Cloud Ribbon 与 Spring Cloud Hystrix
* 提供了声明式的服务调用(不再通过RestTemplate)。

----
# Zuul
> 统一的消费者工程调用入口

* 网关是系统唯一对外的入口，介于客户端与服务器端之间，用于对请求进行鉴权、限流、路由、监控等功能
* 通过与SpringCloud Eureka进行整合，将自身注册为Eureka服务治理下的应用，同时从Eureka中获得了所有其他微服务的实例信息。
* 外层调用都必须通过API网关，使得将维护服务实例的工作交给了服务治理框架自动完成
* 在API网关服务上进行统一调用来对微服务接口做前置过滤，以实现对微服务接口的拦截和校验

Zuul的主要作用
* 路由匹配(动态路由)
* 过滤器实现(动态过滤器)


Zuul 的过滤功能
* 默认会过滤掉Cookie与敏感的HTTP头信息(额外配置)
* 令牌桶限流: 有个桶,如果里面没有满那么就会以一定固定的速率会往里面放令牌，一个请求过来首先要从桶中获取令牌，如果没有获取到，那么这个请求就拒绝，如果获取到那么就放行
* zuul是对外暴露的唯一接口相当于路由的是controller的请求，而Ribbonhe和Fegin路由了service的请求
* zuul做最外层请求的负载均衡 ，而Ribbon和Fegin做的是系统内部各个微服务的service的调用的负载均衡


# SpringCloud Config
> 不应该每个应用下一个一个寻找配置文件然后修改配置文件再重启应用。


简单来说，使用Spring Cloud Config就是将配置文件放到统一的位置管理(比如GitHub)，客户端通过接口去获取这些配置文件

> 如果我在应用运行时去更改远程配置仓库(Git)中的对应配置文件，那么依赖于这个配置文件的已启动的应用不会进行其相应配置的更改呢

---
# Spring Cloud Bus
> 用于将服务和服务实例与分布式消息系统链接在一起的事件总线。

消息代理中间件构建一个共用的消息主题让所有微服务实例订阅，当该消息主题产生消息时会被所有微服务实例监听和消费。

Spring Cloud Bus的作用就是管理和广播分布式系统中的消息，需要把一个操作散发到所有后端相关服务器的时候，就可以选择使用 Spring Cloud Bus 了

我们需要更新配置，又或者需要同时失效所有服务器上的某个缓存，需要向所有相关的服务器发送命令，此时就可以选择使用 Spring Cloud Bus 了


