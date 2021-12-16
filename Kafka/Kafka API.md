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

