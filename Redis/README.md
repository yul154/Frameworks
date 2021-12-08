- Redis
  -  和Mysql的区别
  -  Redis 特点
  -  使用场景
  -  Why Redis is so fast
  -  Redis数据类型
    - 不同类型的底层实现和编码对应关系
    - RedisObject
  - Redis 持久化
    - RDB
    - AOP
  - Redis缓存问题
    - 缓存击穿
    - 缓存雪崩
    - 缓存穿透
    - 数据一致性
    - redis的过期策略以及内存淘汰机制
  - Redis 分布式锁
  - Redis 事务
  - Redis 事件
  - Redis 高可用性
   - 主从复制
   - 哨兵机制
  - Redis 高扩展性
    - 分片


-------
> 为啥用Redis？
> Redis有哪些数据结构呀？
>如果有大量的key需要设置同一时间过期，一般需要注意什么？ —》 一般需要在时间上加一个随机值，使得过期时间分散一些。
> 那你使用过Redis分布式锁么，它是什么回事？
> 使用过Redis做异步队列么，你是怎么用的？
> Redis是怎么持久化的？服务主从数据怎么交互的？
> Pipeline有什么好处，为什么要用pipeline？
> 什么是缓存雪崩？
> 如何保证缓存和数据库数据的一致性？
> Redis的同步机制了解么？
> 是否使用过Redis集群，集群的高可用怎么保证，集群的原理是什么？


