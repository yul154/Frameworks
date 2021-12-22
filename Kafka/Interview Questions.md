> Kafka 是什么？主要应用场景有哪些？



> 和其他消息队列相比,Kafka的优势在哪里？


> 队列模型了解吗？Kafka 的消息模型知道吗？

> 什么是Producer、Consumer、Broker、Topic、Partition？


> Kafka 的多副本机制了解吗？带来了什么好处？

> Zookeeper 在 Kafka 中的作用知道吗？


> Kafka 如何保证消息的消费顺序？


> kafka 如何保证消息不丢失

三类消息

> kafka是怎么保证高可用性的

备份机制



> kafka重平衡，重启服务怎么保证kafka不发生重平衡，有什么方案

cooperative协议将一次全局重平衡，改成每次小规模重平衡，直至最终收敛平衡的过程。
* assigned-partitions和owned-partitions，group coordinator通过这两者，可以保存和获取分区的消费状态，以便进行多次重平衡并达到最终的均衡状态。

static membership
* group.instance.id,一旦配置了该参数，成员将自动成为静态成员，否则的话和以前一样依然被视为是动态成员
* 因此之前分配给该成员的所有分区也是不变的
* 即假设一个成员挂掉，在没有超时前静态成员重启回来是不会触发 Rebalance 的
* 在静态成员挂掉这段时间，broker会一直为该消费者保存状态（offset），直到超时或静态成员重新连接。

第一类非必要 Rebalance 是因为未能及时发送心跳，导致 Consumer 被“踢出”Group 而引发的
* session.timeout.ms 和 heartbeat.interval.ms的

非必要 Rebalance 是 Consumer 消费时间过长导致的
* max.poll.interval.ms参数值的设置显得尤为关键。如果要避免非预期的 Rebalance，你最好将该参数值设置得大一点


> kafka怎么保证高吞吐量的，

支持顺序读写磁盘实现数据存储 ...
支持批量投递和获取消息，减少IO操作 ...
采用零拷贝机制--可减少用户空间的拷贝
采用分区存放消息，根据Partition实现对我们的数据的分区

> .kafka支持事务么

而Kafka 事务消息则是用在一次事务中需要发送多个消息的情况，保证多个消息之间的事务约束，即多条消息要么都发送成功，要么都发送失败
Kafka 的事务基本上是配合其幂等机制来实现Exactly Once 语义的

> 消息可靠性，消息重复消费。如果消息丢失，你应该怎么尽量地让用户觉得此次下单的

消息丢失
* 消费端弄丢了数据: 改成手动
* Kafka 弄丢了数据 
    * 给 topic 设置 replication.factor 参数
    * 在 Kafka 服务端设置 min.insync.replicas 参数
    * 在 producer 端设置 acks=all
    * 在 producer 端设置 retries=MAX

* 生产者会不会弄丢数据？: 按照上述的思路设置了 acks=all，一定不会丢

消息重复
* 消费端处理消息的业务逻辑保持幂等性
* 保证每条消息都有唯一编号且保证消息处理成功与去重表的日志同时出现


