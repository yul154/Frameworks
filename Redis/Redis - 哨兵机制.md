# 哨兵机制
> 哨兵机制是实现主从库自动切换的关键机制，它有效地解决了主从复制模式下故障转移的问题

哨兵实现了什么功能呢
* 监控（Monitoring）：哨兵会不断地检查主节点和从节点是否运作正常。
* 自动故障转移（Automatic failover）：当主节点不能正常工作时，哨兵会开始自动故障转移操作，它会将失效主节点的其中一个从节点升级为新的主节点，并让其他从节点改为复制新的主节点
* 配置提供者（Configuration provider）：客户端在初始化时，通过连接哨兵来获得当前Redis服务的主节点地址。
* 通知（Notification）：哨兵可以将故障转移的结果发送给客户端。


## 哨兵集群的组建
> 哨兵实例之间可以相互发现，要归功于 Redis 提供的 pub/sub 机制


* 在主从集群中，主库上建立频道，
* 哨兵 把自己的 IP和端口发布到_频道上
* 其他哨兵订阅了该频道。那么此时，哨兵就可以从这个频道直接获取哨兵的IP 地址和端口号。
* 然后，哨兵就建立网络连接


## 哨兵监控Redis库
* 哨兵向主库发送`INFO`命令来完成的
* 主库接受到这个命令后，就会把从库列表返回给哨兵
* 哨兵就可以根据从库列表中的连接信息，和每个从库建立连接
* 并在这个连接上持续地对从库进行监控


## 主库下线的判定
```
* 主观下线：任何一个哨兵都是可以监控探测，并作出Redis节点下线的判断；
* 客观下线：有哨兵集群共同决定Redis节点是否下线；
```
* 当某个哨兵判断主库“主观下线”后，就会给其他哨兵发送`is-master-down-by-add` 命令。
* 接着，其他哨兵会根据自己和主库的连接情况,做出 Y 或 N 的响应
* 如果赞成票数是大于等于哨兵配置文件中的`quorum`配置项 则可以判定主库客观下线了

## 哨兵集群的选举

* 主的哨兵节点:故障的转移和通知
* 哨兵的分布式集群: 为了避免哨兵的单点情况发生,作为分布式集群，必然涉及共识问题（即选举问题）

哨兵的选举机制其实很简单，就是一个Raft选举算法:选举的票数大于等于num(sentinels)/2+1时，将成为领导者，如果没有超过，继续选举

任何一个想成为`Leader`的哨兵，要满足两个条件：
* 拿到半数以上的赞成票；
* 拿到的票数同时还需要大于等于哨兵配置文件中的 quorum 值。


## 新主库的选出

* 过滤掉不健康的（下线或断线），没有回复过哨兵ping响应的从节点
* 选择`slave-priority`从节点优先级最高（redis.conf）的
* 选择复制偏移量最大，指复制最完整的从节点

## 故障的转移

* 将slave-1脱离原从节点升级主节点， 
* 将从节点slave-2指向新的主节点 
* 通知客户端主节点已更换 
* 将原主节点（oldMaster）变成从节点，指向新的主节点
