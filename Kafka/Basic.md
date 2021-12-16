MessageQueue
- Why Message Queue
- FrameWork


Kafka 
 - 基本概念
 - 深入构架
  - 文件存储
    - 生产者
      - 分区策略
      - 数据可靠性保证 - Exactly Once
    - 消费者
      - 消费方式
      - 分区分配策略
      - offset 的维护
      - Zookeeper 在 Kafka 中的作用
 -  Kafka 事务

-----
# Message Queue

使用消息队列的好处
* 解耦 : 允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。
* 可恢复性: 系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理
* 缓冲: 有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况
* 灵活性 & 峰值处理能力: 使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃
* 异步通信: 用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们


消息队列的两种模式
1.点对点(Queue，不可重复消费):消息生产者生产消息发送到queue中，然后消息消费者从queue中取出并且消费消息。 
  * 消息被消费以后，queue中不再有存储，所以消息消费者不可能消费到已经被消费的消息。
  * Queue支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费

2.发布/订阅(Topic，可以重复消费):使用topic作为通信载体
  * 消息生产者（发布）将消息发布到topic中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费
  * topic实现了发布和订阅，当你发布一个消息，所有订阅这个topic的服务都能得到这个消息，所以从1到N个订阅者都能得到一个消息的拷贝
 
 
 
## kafka、activemq、rabbitmq、rocketmq都有什么优点和缺点啊？
持久化日志
 * 持久性 ：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失；Kafka 持久化日志，这些日志可以被重复读取和无限期保留
 * 日志收集：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer
多副本存储机制
 * 副本间消息同步、异步复制，数据同步或异步落盘多种方式供您自由选择
 * 可靠性: 为保证集群中的某个节点发生故障时，该节点上的partition数据不丢失，且kafka仍然能够继续工作，kafka提供了副本机制，
高并发
 *  同一个topic分成不同的partition放在不同的broker上

 


----

# Kafka基础架构

<img width="698" alt="Screen Shot 2021-12-15 at 10 12 22 AM" src="https://user-images.githubusercontent.com/27160394/146110381-a86a21e3-14e5-4b37-b5af-fca1351649ec.png">

* Producer：消息生产者，就是向kafka broker发消息的客户端；
* Consumer：消息消费者，向kafka broker取消息的客户端；
  * Consumer Group(CG):由多个consumer组成。消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个消费者消费；消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
  
* Topic: 消息以主题（Topic）来分类，每一个主题都对应一个消息队列,可以有多个生产者往同一个队列(topic)丢数据，多个消费者往同一个队列(topic)拿数据;(相当于数据库里边表的概念)
* Partition: Partition 属于 Topic 的一部分, 一个topic可以分为多个partition
    * 同一 Topic下的Partition 可以分布在不同的 Broker 上.
    * 每个partition是一个有序的队列.

* Broker: 一台kafka服务器就是一个broker,可以看作是一个独立的 Kafka 实例.
    * 一个集群由多个broker组成,其中集群内某个 Broker 会成为集群控制器（Cluster Controller),这个 Broker 也称为这个分区的 Leader
    * 一个broker可以容纳多个topic,
    * 一个topic会分为多个partition，实际上partition会分布在不同的broker中
   
* Replica 副本，为保证集群中的某个节点发生故障时，该节点上的partition数据不丢失，且kafka仍然能够继续工作，kafka提供了副本机制，
  * 一个topic的每个分区都有若干个副本，一个leader和若干个follower
  * Leader 每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是leader。
  * Follower 每个分区多个副本中的“从”，实时从leader中同步数据，保持和leader数据的同步。leader发生故障时，某个follower会成为新的leader
  * 生产者和消费者只与 leader 副本交互


> Kafka 的多分区（Partition）以及多副本（Replica）机制有什么好处呢？

1. Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力（负载均衡）
2. Partition 可以指定对应的 Replica 数, 这也极大地提高了消息存储的安全性, 提高了容灾能力，不过也相应的增加了所需要的存储空间

---
# Kafka 的设计与实现

<img width="565" alt="Screen Shot 2021-12-15 at 10 40 29 AM" src="https://user-images.githubusercontent.com/27160394/146113075-4c945289-6c69-48c8-913b-355cbc4b7d79.png">

* topic: 消息是以 topic 进行分类的，生产者生产消息，消费者消费消息，都是面向 topic 的。
  * topic 是逻辑上的概念,而partition 是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是producer生产的数据,Producer 生产的数据会被不断追加到该 log 文件末端
* offset: 每条数据都有自己的 offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个 offset，以便出错恢复时，从上次的位置继续消费


## Kafka 文件存储

Kafka 的消息是存在于文件系统之上的。Kafka 高度依赖文件系统来存储和缓存消息

<img width="651" alt="Screen Shot 2021-12-15 at 10 42 59 AM" src="https://user-images.githubusercontent.com/27160394/146113321-b20f8375-5975-4f51-a2f5-bf569038dcf4.png">

任何发布到 Partition 的消息都会被追加到 log 文件的尾部，这样的顺序写磁盘操作让 Kafka 的效率非常高
* 为防止log文件过大导致数据定位效率低下，Kafka 采取了分片和索引机制，
  * 将每个 partition 分为多个 segment。
  * 每个 segment 对应两个文件——“.index”文件和 “.log” 文件
  * Segment 是 Kafka 文件存储的最小单位
 
```
| --topic1-0
    | --00000000000000000000.index
    | --00000000000000000000.log
    | --00000000000000368769.index
    | --00000000000000368769.log
    | --00000000000000737337.index
    | --00000000000000737337.log
    | --00000000000001105814.index
    | --00000000000001105814.log
| --topic2-0
| --topic2-1
```
 
数据文件和索引位于一个文件夹下
* 该文件夹的命名规则为：topic 名称 + 分区序号,`first-0,first-1,first-2`
* index和log文件以当前 segment 的第一条消息的 offset 命名。`00000000000000170410.index, 00000000000000170410.log` 
  * log 文件存储大量的数据
   * index 文件存储大量的索引信息.
       * 索引文件中的元数据指向对应数据文件中message的物理偏移地址。 
    
```
因为其文件名为上一个 Segment 最后一条消息的 offset
* 所以当需要查找一个指定 offset 的 message 时，通过在所有 segment 的文件名中进行二分查找就能找到它归属的 segment ，
* 再在其 index 文件中找到其对应到文件上的物理位置，就能拿出该 message 。
```


>  Kafka 是如何准确的知道 message 的偏移的呢？这是因为在 Kafka 定义了标准的数据存储结构，在 Partition 中的每一条 message 都包含了以下三个属性：
>  * offset：表示 message 在当前 Partition 中的偏移量，是一个逻辑上的值，唯一确定了 Partition 中的一条 message，可以简单的认为是一个 id；
>  * MessageSize：表示 message 内容 data 的大小；
>  * data：message 的具体内容

----

## Kafka 生产者

*生产者写消息的基本流程*

1. 创建一个ProducerRecord : 这个对象需要包含消息的主题（topic）和值(value),可以选择性指定一个键值（key）或者分区（partition）。
2. 对这个对象进行序列化 : 因为 Kafka 的消息需要从客户端传到服务端，涉及到网络传输，所以需要实现序列
3. 发送到分配器(partitioner): 如果我们指定了分区，那么分配器返回该分区即可；否则，分配器将会基于键值来选择一个分区并返回。
5. 生产者知道了消息所属的主题和分区，发送这条记录到相同主题和分区的批量消息中(不是直接被发送到服务端，而是放入了生产者的一个缓存里面),在这个缓存里面，多条消息会被封装成为一个批次（batch）
6. `Sender`线程启动以后会从缓存里面去获取可以发送的批次,`Sender`线程把一个一个批次发送到broker
7. 当broker接收到消息后，如果成功写入则返回一个包含消息的主题、分区及位移的RecordMetadata对象，否则返回异常。
8. 生产者接收到结果后，对于异常可能会进行重试。

<img width="583" alt="Screen Shot 2021-12-16 at 10 59 46 AM" src="https://user-images.githubusercontent.com/27160394/146300209-2804c334-a0a2-4e1c-beae-0aa9ba6c3c29.png">

**Spark**
* 消息缓冲区
* `Sender`线程把一个一个批次发送到broker
* 回执确认：acks，如对效率要求较高的情况下建议使用0



----
## Broker

broker 是消息的代理，Producers往Brokers里面的指定Topic中写消息，Consumers从Brokers里面拉取指定Topic的消息，然后进行业务处理，broker在中间起到一个代理保存消息的中转站

### 分区策略

数据存在不同的partition上，那kafka就把这些partition做备份。比如，现在我们有三个partition，分别存在三台broker上。每个partition都会备份，这些备份散落在不同的broker上

分区的原因
* 方便在集群中扩展，每个 Partition 可以通过调整以适应他所在的机器，而一个 topic 可以有多个 Partition 组成，因此这个集群就可以适应任意大小的数据了；
* 可以提高并发，因为可以以 Partition 为单位读写了。
  
分区的原则
* 我们将 producer 发送的数据封装成一个 ProducerRecord 对象。
  * 指明 partition 的情况下，直接将指明的值直接作为 partition 值；
  * 没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 值；
  * 既没有partition值有没有key值的情况下，第一次调用时随机生成一个整数(后面调用在这个整数上自增)，将这个值的 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 Round Robin（轮询调度）算法。
  
 
### 数据可靠性保证(主从复制)


1. 为保证 producer 发送的数据，能可靠的发送到指定的 topic
2. topic 的每个 partition 收到 producer 发送的数据后，都需要向 producer 发送 ack（acknowledgement 确认收到），
3. 如果 producer 收到 ack，就会进行下一轮的发送，否则重新发送数据

<img width="646" alt="Screen Shot 2021-12-15 at 11 38 28 AM" src="https://user-images.githubusercontent.com/27160394/146118778-3012e9c3-a4da-4c10-a673-bb661e0f3b2e.png">


```
副本数据同步策略
1. 半数以上完成同步，就发送 ack	延迟低	选举新的 leader 时，容忍 n 台节点的故障，需要 2n+1 个副本
2. 全部完成同步，才发送 ack	选举新的 leader 时，容忍 n 台节点的故障，需要 n+1 个副本	
```

Kafka 选择了第二种方案，
* 同样为了容忍 n 台节点的故障，第一种方案需要 2n+1 个副本，而第二种方案只需要 n+1 个副本，而 Kafka 的每个分区都有大量的数据，第一种方案会造成大量数据的冗余
* 虽然第二种方案的网络延迟会比较高，但网络延迟对 Kafka 的影响较小（同一网络环境下的传输）


* ISR：in-sync replics，每个分区(Partition)中同步的副本列表。
* Hight Watermark：副本水位值，表示分区中最新一条已提交(Committed)的消息的Offset。
* LEO：Log End Offset，Leader中最新消息的Offset。
* Committed Message：已提交消息，已经被所有ISR同步的消息。
* Lagging Message：没有达到所有ISR同步的消息。


ISR
> 有一个 follower，因为某种故障，迟迟不能与 leader 进行同步
* Leader 维护了一个动态的 in-sync replica set (ISR)，意为和 leader 保持同步的 follower 集合。
* 当 ISR 中的 follower 完成数据的同步之后，leader 就会给 producer 发送 ack。
* 如果 follower 长时间未向 leader 同步数据，则该 follower 将被踢出 ISR，该时间阈值由 replica.lag.time.max.ms 参数设定。
* Leader 发生故障之后，就会从 ISR 中选举新的 leader

ack 应答机制
> 对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等 ISR 中的 follower 全部接收成功。

* ack = 0 : producer 不等待 broker 的 ack，这一操作提供了一个最低的延迟，broker 一接收到还没有写入磁盘就已经返回，当 broker 故障时有可能丢失数据
* ack = 1 : producer 等待 broker 的 ack，partition 的 leader 落盘成功后返回 ack，如果在 follower 同步成功之前 leader 故障，那么就会丢失数据
* ack = -1(all)：producer 等待 broker 的 ack，partition 的 leader 和 follower（是 ISR 中的） 全部落盘成功后才返回 ack，但是如果 follower 同步完成后，broker 发送 ack 之前，leader 发生故障，producer 重新发送消息给新 leader 那么会造成数据重复。

```
数据只要被Leader写入log就被认为已经commit

follower 故障 follower 发生故障后会被临时踢出 ISR
待该 follower 恢复后，follower 会读取本地磁盘记录的上次的 HW，并将 log 文件高于 HW 的部分截取掉，从 HW 开始向 leader 进行同步。
等该 follower 的 LEO 大于等于该 Partition 的 HW，即 follower 追上 leader 之后，就可以重新加入 ISR 了。

leader 故障 leader 发生故障之后，会从 ISR 中选出一个新的 leader，之后，
为保证多个副本之间的数据一致性，其余的 follower 会先将各自的log文件高于 HW 的部分截掉，然后从新的 leader 同步数据。
````

####  Exactly Once



|寓意|级别|优劣势|
|---|----|-----|
|At Least Once| 将服务器的 ACK 级别设置为 -1，可以保证 Producer 到 Server 之间不会丢失数据| 可以保证数据不丢失，但是不能保证数据不重复；|
|At Most Once|将服务器 ACK 级别设置为 0，可以保证生产者每条消息只会被 发送一次|可以保证数据不重复，但是不能保证数据不丢失|


**幂等性**
> 对于一些非常重要的信息，比如说交易数据，下游数据消费者要求数据既不重复也不丢失，即 Exactly Once 语义。
* 所谓的幂等性就是指 Producer 不论向 Server 发送多少次重复数据，Server 端都只会持久化一条。幂等性结合 At Least Once 语义，就构成了 Kafka 的 Exactly Once 
* At Least Once + 幂等性 = Exactly Once 
* 要启用幂等性，只需要将 Producer 的参数中`enable.idompotence`设置为 true
* Kafka 的幂等性实现其实就是将原来下游需要做的去重放在了数据上游
* 开启幂等性的Producer在初始化的时候会被分配一个`PID`，发往同一`Partition`的消息会附带 Sequence Number。而 Broker端会对做缓存，当具有相同主键的消息提交时，Broker 只会持久化一条
* PID重启就会变化，同时不同的 Partition 也具有不同主键，所以幂等性无法保证跨分区跨会话的 Exactly Once

----
#  Kafka 消费者

## 消费方式
> Kafka consumer 采用`pull`模式从 broker 中读取数据。


* push : 从Broker 推向Consumer，即Consumer 被动的接收消息,由Broker 来主导消息的发送
  * push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker 决定的。
  * 它的目标是尽可能以最快速度传递消息，但是这样很容易造成 consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。
  * 
* pull : 指的是 Consumer 主动向 Broker 请求拉取消息，即 Broker 被动的发送消息给 Consumer
  * pull 模式则可以根据 consumer 的消费能力以适当的速率消费消息。
  * pull 模式可简化 broker 的设计
  * 如果 kafka 没有数据，消费者可能会陷入循环中，一直返回空数据
    * Kafka 的消费者在消费数据时会传入一个时长参数`timeout`，如果当前没有数据可供消费，consumer 会等待一段时间之后再返回，这段时长即为 timeout。



## 分区分配策略
> 哪个 partition 由哪个 consumer 来消费

Kafka 有两种分配策略，
* RoundRobin : 根据 partition 号对 consumer 个数取模后轮循分配
<img width="203" alt="Screen Shot 2021-12-15 at 12 39 54 PM" src="https://user-images.githubusercontent.com/27160394/146124332-f2ed4163-7143-47a0-a40e-ebe78916589f.png">

* range : 提前按照均匀分配的原则计算个数后直接分配。

<img width="240" alt="Screen Shot 2021-12-15 at 12 40 33 PM" src="https://user-images.githubusercontent.com/27160394/146124398-23c2bad6-7ede-4877-9ee7-8f368d171ab6.png">


> 在订阅多个 partition 时 range 会有不均匀问题，kafka 默认为 range，因为不考虑多 partition 订阅时，range 效率更高。


#### 分区重平衡
> 当消费者离开消费组（比如重启、宕机等）时，它所消费的分区会分配给其他分区。这种现象称为重平衡(rebalance)

在重平衡期间，所有消费者都不能消费消息，因此会造成整个消费组短暂的不可用


消费者通过定期发送心跳（hearbeat）到一个作为组协调者(group coordinator)的broker来保持在消费组内存活。
* 这个 broker 不是固定的，每个消费组都可能不同
* 当消费者拉取消息或者提交时，便会发送心跳。
* 如果消费者超过一定时间没有发送心跳，那么它的会话（session）就会过期，组协调者会认为该消费者已经宕机，然后触发重平衡。
* 从消费者宕机到会话过期是有一定时间的，这段时间内该消费者的分区都不能进行消息消费；
  *  通常情况下，我们可以进行优雅关闭，这样消费者会发送离开的消息到组协调者，这样组协调者可以立即进行重平衡而不需要等待会话过期
  *  将发送心跳与拉取消息进行分离，这样使得发送心跳的频率不受拉取的频率影响
  *  一个消费者多长时间不拉取消息但仍然保持存活，这个配置可以避免活锁（livelock）(活锁，是指应用没有故障但是由于某些原因不能进一步消费)



### Partition 被消费

>  Consumer Group 在消费时需要从不同的 Partition 获取消息，那最终如何重建出 Topic 中消息的顺序呢?

没有办法。Kafka 只会保证在 Partition 内消息是有序的

> Partition 中的消息可以被（不同的 Consumer Group）多次消费，那 Partition中被消费的消息是何时删除的？Partition 又是如何知道一个 Consumer Group 当前消费的位置呢？

无论消息是否被消费，除非消息到期 Partition 从不删除消息。例如设置保留时间为 2 天，则消息发布 2 天内任何 Group 都可以消费，2 天后，消息自动被删除。
Partition 会为每个 Consumer Group 保存一个偏移量，记录 Group 消费到的位置




## offset 的维护
> 由于consumer在消费过程中可能会出现断电宕机等故障,consumer恢复后需要从故障前的位置的继续消费,所以 consumer 需要实时记录自己消费到了哪个 offset，以便故障恢复后继续消费

同一个消费者组中的消费者， 同一时刻只能有一个消费者消费。

group + topic + partition（GTP） 才能确定一个 offset

1. 修改配置文件`consumer.properties`
```
exclude.internal.topics=false
```
2. 读取offset
```
bin/kafkabin/kafka--consoleconsole--consumer.sh consumer.sh ----topic __consumer_offsets topic __consumer_offsets ----zookeeper zookeeper hadoophadoop102102:2181 :2181 ----formatter formatter
```


-----
# Kafka 性能

## 消息高可靠性(处理消息丢失的问题)

1. Producer在发布消息到某个Partition时，先通过ZooKeeper找到该Partition的Leader，然后无论该Topic的Replication Factor为多少（也即该Partition有多少个Replica），
2. Producer只将该消息发送到该Partition的Leader。
3. Leader会将该消息写入其本地Log。每个Follower都从Leader中pull数据

### 生产者丢数据

不使用`producer.send(msg)`，而使用带回调的`producer.send(msg, callback)`方法；

1. 配置要在producer端设置acks=all。这个配置保证了，follwer同步完成后，才认为消息发送成功。
2. producer端设置retries=MAX，一旦写入失败，这无限重试

### 消息队列丢数据
> 数据还没同步，leader就挂了，这时zookpeer会将其他的follwer切换为leader,那数据就丢失了

* `replication.factor`参数，这个值必须大于1，即要求每个partition必须有至少2个副本
* `min.insync.replicas`参数，这个值必须大于1，这个是要求一个leader至少感知到有至少一个follower还跟自己保持联系
* 确保`replication.factor` > `min.insync.replicas`。若两者相等，则如果有一个副本挂了，整个分区就无法正常工作了。推荐设置为：replication.factor = min.insync.replicas + 1；


### 消费者丢数据

消费端数据丢失的原因是 offset 的自动提交。
* 消费者会自动每隔一段时间将offset保存到zookeeper上，此时如果刚好将偏移量提交到zookeeper上后，但这条数据还没消费完，机器发生宕机，此时数据就丢失了

改成手动提交即可: 关闭自动提交，改成手动提交，每次数据处理完后，再提交。


Kafka消息消费有两个consumer接口，Low-level API和High-level API：

* Low-level API：消费者自己维护offset等值，可以实现对Kafka的完全控制；
* High-level API：封装了对parition和offset的管理，使用简单


### 如何保证消息不被重复消费？


Kafka所提供的消息精确一次消费的手段有两个：幂等性Producer和事务型Producer。
* 幂等性Producer只能保证单会话、单分区上的消息幂等性；
* 事务型Producer可以保证跨分区、跨会话间的幂等性；
* 事务型Producer功能更为强大，但是同时，其效率也会比较低下。



目前Kafka默认提供的消息可靠机制是“至少一次” :

Exactly once: At Least Once + 幂等性 
* 幂等性就是指 Producer 不论向 Server 发送多少次重复数据，Server 端都只会持久化一条。幂等性结合 At Least Once 语义，就构成了 Kafka 的 Exactly Once 
* Kafka 的幂等性实现其实就是将原来下游需要做的去重放在了数据上游
  * 要启用幂等性，只需要将 Producer 的参数中`enable.idompotence`设置为 true
  * 开启幂等性的Producer在初始化的时候会被分配一个`PID`，发往同一`Partition`的消息会附带 Sequence Number。而 Broker端会对做缓存，当具有相同主键的消息提交时，Broker 只会持久化一条
  * PID重启就会变化，同时不同的 Partition 也具有不同主键，所以幂等性无法保证跨分区跨会话的 Exactly Once


幂等性并不能跨多个分区运作，而Kafka事务则可以弥补这个缺陷,主要在read committed隔离级别
* 设置Producer端参数transcational.id。最好为其设置一个有意义的名字。
* 事务型Producer可以保证record1和record2要么全部提交成功，要么全部写入失败
* 读取事务型Producer发送的消息时，Consumer端的isolation.level参数表征着事务的隔离级别，即决定了Consumer以怎样的级别去读取消息
  * read_uncommitted：默认值，表面Consumer能够读到Kafka写入的任何消息，不论事务型Producer是否正常提交了事务。显然，如果启用了事务型的Producer，则Consumer端参数就不要使用该值，否则事务是无效的。
  * read_committed：表面Consumer只会读取事务型Producer成功提交的事务中写入的消息，同时，非事务型Producer写入的所有消息对Consumer也是可见的。



### 如何保证消息的顺序性？

* kafka每个partition中的消息在写入时都是有序的，消费时，每个partition只能被每一个group中的一个消费者消费，保证了消费时也是有序的。
* 整个topic不保证有序。如果为了保证topic整个有序，那么将partition调整为1.


## kafka 的高可用机制


## Kafka 高效读写数据（谈谈 Kafka 吞吐量为何如此高）

### 1.顺序写磁盘
Kafka 的 producer 生产数据，要写入到 log 文件中，写的过程是一直追加到文件末端，为顺序写。官网有数据表明，同样的磁盘，顺序写能到到 600M/s，而随机写只有 100k/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。

### 2. 零拷贝技术

零拷贝主要的任务就是避免 CPU 将数据从一块存储拷贝到另外一块存储，主要就是利用各种零拷贝技术，避免让 CPU 做大量的数据拷贝任务，减少不必要的拷贝，或者让别的组件来做这一类简单的数据传输任务，让 CPU 解脱出来专注于别的任务。这样就可以让系统资源的利用更加有效。

### 3.分区分段+索引

* Kafka的message是按topic分类存储的，topic中的数据又是按照一个一个的partition即分区存储到不同broker节点。
* 每个partition对应了操作系统上的一个文件夹，partition实际上又是按照segment分段存储的。这也非常符合分布式系统分区分桶的设计思想


#### 4.批量读写
Kafka数据读写也是批量的而不是单条的


## Zookeeper 在 Kafka 中的作用

kafka在所有broker中选出一个controller，所有Partition的Leader选举都由controller决定。
* controller会将Leader的改变直接通过RPC的方式（比Zookeeper Queue的方式更高效）通知需为此作出响应的Broker。
* 同时controller也负责增删Topic以及Replica的重新分配。

Zookeeper
* Kafka 集群中有一个 broker 会被选举为 Controller，负责管理集群 broker 的上下线，所有 topic 的分区副本分配和 leader 选举等工作。
* Controller 的管理工作都是依赖于 Zookeeper 的。

<img width="550" alt="Screen Shot 2021-12-15 at 12 49 47 PM" src="https://user-images.githubusercontent.com/27160394/146125167-419fa276-b675-4263-85eb-46e6886364ad.png">


---

# Kafka 事务
> 事务可以保证 Kafka 在 Exactly Once 语义的基础上，生产和消费可以跨分区和会话，要么全部成功，要么全部失败。

* Producer事务事务
```
为了实现跨分区跨会话的事务，需要引入一个全局唯一的 Transaction ID（一定是客户端给的），并将 Producer 获得的 PID 和 Transaction ID 绑定。这样当 Producer 重启后就可以通过正在进行的 Transaction ID 获得原来的 PID
```

* Consumer 事务
```
对于 Consumer 而言，事务的保证就会相对较弱，尤其时无法保证 Commit 的信息被精确消费。这是由于 Consumer 可以通过 offset 访问任意信息，而且不同的 Segment File 生命周期不同，同一事务的消息可能会出现重启后被删除的情况。
```



