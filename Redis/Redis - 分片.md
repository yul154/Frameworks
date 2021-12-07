# Redis Cluster


## 主要模块介绍

### 1.哈希槽(Hash Slot)
Redis-cluster中有16384(即2的14次方）个哈希槽，每个key通过CRC16校验后对16383取模来决定放置哪个槽.

Cluster中的每个节点负责一部分hash槽(hash slot),比如集群中存在三个节点
* 节点A包含0到5500号哈希槽；
* 节点B包含5501到11000号哈希槽；
* 节点C包含11001 到 16384号哈希槽

**Keys hash tags**
* Hash tags提供了一种途径，用来将多个(相关的)key分配到相同的hash slot中。这时Redis Cluster中实现multi-key操作的基础
* 
