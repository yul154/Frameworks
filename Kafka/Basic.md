MessageQueue
- Kafka 基本构架
  - 深入构架
    - 文件存储
    - 生产者
      - 分区策略
      - 数据可靠性保证 - Exactly Once
    - 消费者
      - 
- Kafka 单点配置 

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
 
 
**Kafka的优势**
* 极致的性能 ：基于 Scala 和 Java 语言开发，设计中大量使用了批量处理和异步的思想，最高可以每秒处理千万级别的消息。
* 生态系统兼容性无可匹敌 ：Kafka 与周边生态系统的兼容性是最好的没有之一，尤其在大数据和流计算领域。



----
# Kafka基础架构



<img width="698" alt="Screen Shot 2021-12-15 at 10 12 22 AM" src="https://user-images.githubusercontent.com/27160394/146110381-a86a21e3-14e5-4b37-b5af-fca1351649ec.png">

* Producer：消息生产者，就是向kafka broker发消息的客户端；
* Consumer：消息消费者，向kafka broker取消息的客户端；
* Consumer Group(CG)：消费者组，由多个consumer组成。消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个消费者消费；消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
* Broker: 一台kafka服务器就是一个broker 可以看作是一个独立的 Kafka 实例.多个 Kafka Broker 组成一个 Kafka Cluster,一个集群由多个broker组成。一个broker可以容纳多个topic,可以看作是一个独立的 Kafka 实例。
* Topic: 可以理解为一个队列，生产者和消费者面向的都是一个topic；
* Partition: 一个topic可以分为多个partition，同一 Topic 下的 Partition 可以分布在不同的 Broker 上.一个非常大的topic可以分布到多个broker（即服务器）上. 每个partition是一个有序的队列.
* Replica 副本，为保证集群中的某个节点发生故障时，该节点上的partition数据不丢失，且kafka仍然能够继续工作，kafka提供了副本机制，一个topic的每个分区都有若干个副本，一个leader和若干个follower
  * Leader 每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是leader。
  * Follower 每个分区多个副本中的“从”，实时从leader中同步数据，保持和leader数据的同步。leader发生故障时，某个follower会成为新的followe

---
# 架构深入

## Kafka 文件存储

<img width="965" alt="Screen Shot 2021-12-15 at 10 40 29 AM" src="https://user-images.githubusercontent.com/27160394/146113075-4c945289-6c69-48c8-913b-355cbc4b7d79.png">

* topic: 消息是以 topic 进行分类的，生产者生产消息，消费者消费消息，都是面向 topic 的。
  * topic 是逻辑上的概念,而partition 是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是producer生产的数据,Producer 生产的数据会被不断追加到该 log 文件末端
* offset: 每条数据都有自己的 offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个 offset，以便出错恢复时，从上次的位置继续消费

<img width="651" alt="Screen Shot 2021-12-15 at 10 42 59 AM" src="https://user-images.githubusercontent.com/27160394/146113321-b20f8375-5975-4f51-a2f5-bf569038dcf4.png">

由于生产者生产的消息会不断追加到 log 文件末尾，为防止 log 文件过大导致数据定位效率低下，Kafka 采取了分片和索引机制，
  * 将每个 partition 分为多个 segment。
  * 每个 segment 对应两个文件——“.index”文件和 “.log” 文件。
  * 这些文件位于一个文件夹下，该文件夹的命名规则为：topic 名称 + 分区序号,`first-0,first-1,first-2`
  * index和log文件以当前 segment 的第一条消息的 offset 命名。`00000000000000170410.index, 00000000000000170410.log`
     * log 文件存储大量的数据
     * index 文件存储大量的索引信息.
         * 索引文件中的元数据指向对应数据文件中message 的物理偏移地址。 


## Kafka 生产者

### 分区策略

分区的原因
* 方便在集群中扩展，每个 Partition 可以通过调整以适应他所在的机器，而一个 topic 可以有多个 Partition 组成，因此这个集群就可以适应任意大小的数据了；
* 可以提高并发，因为可以以 Partition 为单位读写了。
  
分区的原则
* 我们将 producer 发送的数据封装成一个 ProducerRecord 对象。
  * 指明 partition 的情况下，直接将指明的值直接作为 partition 值；
  * 没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 值；
  * 既没有partition值有没有key值的情况下，第一次调用时随机生成一个整数(后面调用在这个整数上自增)，将这个值的 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 Round Robin（轮询调度）算法。
  
 
### 数据可靠性保证

* ISR：in-sync replics，每个分区(Partition)中同步的副本列表。
* Hight Watermark：副本水位值，表示分区中最新一条已提交(Committed)的消息的Offset。
* LEO：Log End Offset，Leader中最新消息的Offset。
* Committed Message：已提交消息，已经被所有ISR同步的消息。
* Lagging Message：没有达到所有ISR同步的消息。



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

---
# Kafka 消费者

## 消费方式
> consumer 采用`pull`模式从 broker 中读取数据。


* push : 从Broker 推向Consumer，即Consumer 被动的接收消息,由Broker 来主导消息的发送
  * push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker 决定的。
  * 它的目标是尽可能以最快速度传递消息，但是这样很容易造成 consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。
  * 
* pull : 指的是 Consumer 主动向 Broker 请求拉取消息，即 Broker 被动的发送消息给 Consumer
  * pull 模式则可以根据 consumer 的消费能力以适当的速率消费消息。
  * 如果 kafka 没有数据，消费者可能会陷入循环中，一直返回空数据
    * Kafka 的消费者在消费数据时会传入一个时长参数`timeout`，如果当前没有数据可供消费，consumer 会等待一段时间之后再返回，这段时长即为 timeout。

## 分区分配策略
哪个 partition 由哪个 consumer 来消费
* 一个 consumer group 中有多个 consumer
*  一个 topic 有多个 partition


Kafka 有两种分配策略，
* RoundRobin : 根据 partition 号对 consumer 个数取模后轮循分配
<img width="203" alt="Screen Shot 2021-12-15 at 12 39 54 PM" src="https://user-images.githubusercontent.com/27160394/146124332-f2ed4163-7143-47a0-a40e-ebe78916589f.png">

* range : 提前按照均匀分配的原则计算个数后直接分配。

<img width="240" alt="Screen Shot 2021-12-15 at 12 40 33 PM" src="https://user-images.githubusercontent.com/27160394/146124398-23c2bad6-7ede-4877-9ee7-8f368d171ab6.png">



> 在订阅多个 partition 时 range 会有不均匀问题，kafka 默认为 range，因为不考虑多 partition 订阅时，range 效率更高。


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

## Kafka 高效读写数据
### 1. 顺序写磁盘
Kafka 的 producer 生产数据，要写入到 log 文件中，写的过程是一直追加到文件末端，为顺序写。官网有数据表明，同样的磁盘，顺序写能到到 600M/s，而随机写只有 100k/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。


### 2. 零拷贝技术

零拷贝主要的任务就是避免 CPU 将数据从一块存储拷贝到另外一块存储，主要就是利用各种零拷贝技术，避免让 CPU 做大量的数据拷贝任务，减少不必要的拷贝，或者让别的组件来做这一类简单的数据传输任务，让 CPU 解脱出来专注于别的任务。这样就可以让系统资源的利用更加有效。


## Zookeeper 在 Kafka 中的作用

* Kafka 集群中有一个 broker 会被选举为 Controller，负责管理集群 broker 的上下线，所有 topic 的分区副本分配和 leader 选举等工作。
* Controller 的管理工作都是依赖于 Zookeeper 的。

<img width="550" alt="Screen Shot 2021-12-15 at 12 49 47 PM" src="https://user-images.githubusercontent.com/27160394/146125167-419fa276-b675-4263-85eb-46e6886364ad.png">


## Kafka 事务
> 事务可以保证 Kafka 在 Exactly Once 语义的基础上，生产和消费可以跨分区和会话，要么全部成功，要么全部失败。

* Producer事务事务
```
为了实现跨分区跨会话的事务，需要引入一个全局唯一的 Transaction ID（一定是客户端给的），并将 Producer 获得的 PID 和 Transaction ID 绑定。这样当 Producer 重启后就可以通过正在进行的 Transaction ID 获得原来的 PID
```

* Consumer 事务
```
对于 Consumer 而言，事务的保证就会相对较弱，尤其时无法保证 Commit 的信息被精确消费。这是由于 Consumer 可以通过 offset 访问任意信息，而且不同的 Segment File 生命周期不同，同一事务的消息可能会出现重启后被删除的情况。
```
---
# Kafka API

## Producer API

**消息发送流程**
* Kafka 的 Producer 发送消息采用的是异步发送的方式。
* 在消息发送的过程中，涉及到线程
  * main 线程
  * Sender 线程
  * 一个线程共享变量——RecordAccumulator（接收器）。
* main 线程将消息发送给 RecordAccumulator，Sender 线程不断从 RecordAccumulator 中拉取消息发送到 Kafka broker

<img width="561" alt="Screen Shot 2021-12-15 at 1 05 51 PM" src="https://user-images.githubusercontent.com/27160394/146126724-91d4893b-ce57-4d88-901f-8aa9283c15c4.png">

### 异步发送 API
```
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>0.11.0.0</version>
</dependency>

```

* `KafkaProducer` 需要创建一个生产者对象，用来发送数据
* `ProducerConfig` 获取所需的一系列配置参数
* `ProducerRecord` 每条数据都要封装成一个 ProducerRecord 对象

1. 不带回调函数的异步（AsyncProducer）
```
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
        "hadoop101:9092,hadoop102:9092,hadoop103:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
        StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
        StringSerializer.class.getName());
props.put(ProducerConfig.ACKS_CONFIG, "all");
props.put(ProducerConfig.RETRIES_CONFIG, 1);
props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
// 配置拦截器

// 通过配置创建 KafkaProducer 对象
KafkaProducer<String, String> producer = new KafkaProducer<>(props);
for (int i = 0; i < 1000; i++) {
    ProducerRecord<String, String> record = new ProducerRecord<>("first", "message" + i);
    producer.send(record);
}
producer.close();
}
```
2. 带回调函数的异步（CallbackProducer）

```
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
        "192.168.72.133:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
        StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
        StringSerializer.class.getName());
props.put(ProducerConfig.ACKS_CONFIG, "all");
props.put(ProducerConfig.RETRIES_CONFIG, 1);
KafkaProducer<String, String> producer = new KafkaProducer<>(props);

for (int i = 0; i < 1000; i++) {
    ProducerRecord<String, String> record = new ProducerRecord<>("first", "message" + i);
    producer.send(record, new Callback() {
        @Override
        public void onCompletion(RecordMetadata recordMetadata, Exception e) {
            if (e == null)
                System.out.println("success:" + recordMetadata.topic() +
                        "-" + recordMetadata.partition() +
                        "-" + recordMetadata.offset());
            else e.printStackTrace();
        }
    });

  }
  producer.close();
}
```


### 同步发送 API
同步发送（SyncProducer）
```
 Properties props = new Properties();
// 添加配置
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop101:9092,hadoop102:9092,hadoop103:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.ACKS_CONFIG, "all");
props.put(ProducerConfig.RETRIES_CONFIG, 1); // 重试次数
props.put(ProducerConfig.LINGER_MS_CONFIG, 500);
// 通过已有配置创建 kafkaProducer 对象
KafkaProducer<String, String> producer = new KafkaProducer<>(props);
// 循环调用 send 方法不断发送数据
for (int i = 0; i < 100; i++) {
    ProducerRecord<String, String> record = new ProducerRecord<>("first", "message" + i);
    RecordMetadata metadata = producer.send(record).get();// 通过 get()方法实现同步效果
    if (metadata != null)
        System.out.println("success:" + metadata.topic() + "-" +
                metadata.partition() + "-" + metadata.offset());
}
producer.close(); // 关闭生产者对象
}
```

## Consumer API

offset 的维护

### 1.自动提交 offset
```
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>0.11.0.0</version>
</dependency>
```

* `KafkaConsumer`：需要创建一个消费者对象，用来消费数据
* `ConsumerConfig`：获取所需的一系列配置参数
* `ConsuemrRecord`：每条数据都要封装成一个 ConsumerRecord 对象

为了使我们能够专注于自己的业务逻辑，Kafka 提供了自动提交 offset 的功能。
自动提交 offset 的相关参数：
* `enable.auto.commit`：是否开启自动提交 offset 功能\
* `auto.commit.interval.ms`：自动提交 offset 的时间间隔

```
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG,"earliest");
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG,true); // 自动提交
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("first"));
```

### 2.手动提交 offset
手动提交 offset 的方法有两种
* 分别是 commitSync（同步提交）和 commitAsync（异步提交）
* 两者的相同点是，都会将本次 poll 的一批数据最高的偏移量提交; 不同点是，commitSync 阻塞当前线程，一直到提交成功，并且会自动失败充实（由不可控因素导致，也会出现提交失败）; 
* 而 commitAsync 则没有失败重试机制，故有可能提交失败。

#### 同步提交 commitSync offset
```
props.put("enable.auto.commit", "false");// 关闭自动提交 offset
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("first"));// 消费者订阅主题
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);// 消费者拉取数据
    for (ConsumerRecord<String, String> record : records) {
        System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
    }
    consumer.commitSync();// 同步提交，当前线程会阻塞直到 offset 提交成功
}
```

#### 异步提交 commitAsync offset
```
while (true) {
      ConsumerRecords<String, String> records = consumer.poll(100);
      for (ConsumerRecord<String, String> record : records) {
          System.out.println("offset:" + record.offset() +
                  "key:" + record.key() + "value" + record.value());
      }
      consumer.commitAsync(new OffsetCommitCallback() {
          public void onComplete(Map<TopicPartition, OffsetAndMetadata> map, Exception e) {
              if (e != null)
                  System.out.println("commit failed for" + map);
          }
      });// 异步提交
  }
    }
```

#### 数据漏消费和重复消费分析
* 先提交 offset 后消费，有可能造成数据的漏消费；
* 先消费后提交 offset，有可能会造成数据的重复消费

Rebalance: 当有新的消费者加入消费者组、已有的消费者推出消费者组或者所订阅的主题的分区发生变化，就会触发到分区的重新分配，重新分配的过程
* 消费者发生 Rebalance 之后，每个消费者消费的分区就会发生变化。
* 消费者要首先获取到自己被重新分配到的分区，并且定位到每个分区最近提交的 offset 位置继续消费
* 要实现自定义存储 offset，需要借助 `ConsumerRebalanceListener`


## 自定义 Interceptor

### 拦截器
* 主要用于实现 clients 端的定制化控制逻辑
*  对于 producer 而言，interceptor 使得用户在消息发送前以及 producer 回调逻辑前有机会对消息做一些定制化需求，比如修改消息等。
*  同时，producer 允许用户指定多个 interceptor 按序作用于同一条消息从而形成一个拦截链(interceptor chain)


Intercetpor 的实现接口`org.apache.kafka.clients.producer.ProducerInterceptor`
* `configure (configs)` 获取配置信息和初始化数据时调用。
* `onSend (ProducerRecord)`: 封装进 KafkaProducer.send 方法中,它运行在用户主线程中。Producer 确保在消息被序列化以及计算分区前调用该方法。用户可以在该方法中对消息做任何操作，但最好保证不要修改消息所属的 topic 和分区，否则会影响目标分区的计算
* `onAcknowledgement (RecordMetadata, Exception)` : 该方法会在消息从 RecordAccumulator 成功发送到 Kafka Broker 之后，或者在发送过程中失败时调用
* `close`： 关闭 interceptor
---
# Kafka 单节点部署

启动 Zookeeper 服务（可选择，自带或是独立的 zk 服务）
```
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```

启动 Kafka 服务
```
bin/kafka-server-start.sh config/server.properties
```

创建 Topic
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
查看 Topic
```
bin/kafka-topics.sh --list --zookeeper localhost:2181
```

产生消息
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

```

消费消息
```
bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic test
```

查看描述 Topic 信息
```
[root@localhost kafka_2.11-1.0.0]# bin/kafka-topics.sh --describe --zookeeper 
Topic:test      PartitionCount:1        ReplicationFactor:1     Configs: // 第一行给出了所有分区的摘要，每个附加行给出了关于一个分区的信息。 由于我们只有一个分区，所以只有一行。

Topic: test     Partition: 0    Leader: 1       Replicas: 1     Isr: 1
```
* Leader: 是负责给定分区的所有读取和写入的节点。 每个节点将成为分区随机选择部分的领导者。
* Replicas: 是复制此分区日志的节点列表，无论它们是否是领导者，或者即使他们当前处于活动状态。
* Isr : 是一组 “同步” 副本。这是复制品列表的子集，当前活着并被引导到领导者。

----

