<img width="573" alt="Screen Shot 2021-12-06 at 6 03 52 PM" src="https://user-images.githubusercontent.com/27160394/144826875-74ed825f-cb13-4de9-97ff-7d2c4ddab442.png">

# Redis 底层实现

* 简单动态字符串 - sds
* 压缩列表 - ZipList
* 快表 - QuickList
* 字典/哈希表 - Dict
* 整数集 - IntSet
* 跳表 - ZSkipList

## 简单动态字符串 - sds
> 这是一种用于存储二进制数据的一种结构, 具有动态扩容的特点.

<img width="330" alt="Screen Shot 2021-12-06 at 6 50 35 PM" src="https://user-images.githubusercontent.com/27160394/144833438-c5078590-5000-4e61-9502-d0f114a1eea2.png">

* sdshdr是头部
* buf是真实存储用户数据的地方. 在buf中, 用户数据后总跟着一个\0. "数据" + "\0"是为所谓的buf

**`sdshdr`**
* len 保存了SDS保存字符串的长度
* buf[] 数组用来保存字符串的每个元素 
* alloc分别以uint8, uint16, uint32, uint64表示整个SDS, 除过头部与末尾的\0, 剩余的字节数. 
* flags 始终为一字节, 以低三位标示着头部的类型, 高5位未使用.

<img width="567" alt="Screen Shot 2021-12-06 at 6 51 14 PM" src="https://user-images.githubusercontent.com/27160394/144833530-d421367c-6fac-4265-aa73-da451811ff0a.png">

### SDS特点
* 常数复杂度获取字符串长度：获取 SDS 字符串的长度只需要读取 len 属性(`strlen key`)
* 杜绝缓冲区溢出: 在进行字符修改的时候，会首先根据记录的 len 属性检查内存空间是否满足需求，如果不满足，会进行相应的空间扩展(`strcat`函数来进行两个字符串的拼接)
* 减少修改字符串的内存重新分配次数:由于len属性和alloc属性的存在，对于修改字符串SDS实现了空间预分配和惰性空间释放
  * 空间预分配：对字符串进行空间扩展的时候，扩展的内存比实际需要的多，这样可以减少连续执行字符串增长操作所需的内存重分配次数。
  * 惰性空间释放：对字符串进行缩短操作时，程序不立即使用内存重新分配来回收缩短后多余的字节，而是使用 alloc 属性将这些字节的数量记录下来，等待后续使用

* 二进制安全:所有SDS的API都是以处理二进制的方式来处理`buf`里面的元素,并且 SDS 不是以空字符串来判断是否结束，而是以 len 属性表示的长度来判断字符串是否结束

> 空间预分配补进一步理解

当执行追加操作时，比如现在给key=‘Hello World’的字符串后追加‘ again!’则这时的len=18，free由0变成了18，
* 为减少修改字符串带来的内存重分配次数，sds采用了“一次管够”的策略
  * 若修改之后sds长度小于1MB,则多分配现有len长度的空间
  * 若修改之后sds长度大于等于1MB，则扩充除了满足修改之后的长度外，额外多1MB空间
* 所以下次还有append追加的时候，如果预分配空间足够，就无须在进行空间分配了。

> 这种分配策略会浪费内存资源吗？

执行过APPEND 命令的字符串会带有额外的预分配空间，这些预分配空间不会被释放，除非该字符串所对应的键被删除，或者等到关闭Redis 之后，再次启动时重新载入的字符串对象将不会有预分配空间
* 因为执行APPEND 命令的字符串键数量通常并不多，占用内存的体积通常也不大，所以这一般并不算什么问题
* 如果执行APPEND 操作的键很多，而字符串的体积又很大的话，那可能就需要修改Redis 服务器，让它定时释放一些字符串键的预分配空间，从而更有效地使用内存

----
## 压缩列表 - ZipList
> 它是由连续的内存块组成的

ziplist是为了提高存储效率而设计的一种特殊编码的双向链表。
* 它可以存储字符串或者整数,存储整数时是采用整数的二进制而不是字符串形式存储。
* 他能在O(1)的时间复杂度下完成list两端的push和pop操作。但是因为每次操作都需要重新分配ziplist的内存，所以实际复杂度和ziplist的内存使用量相关。

<img width="463" alt="Screen Shot 2021-12-06 at 10 40 23 PM" src="https://user-images.githubusercontent.com/27160394/144865629-c46bbc9d-73f0-4d8b-ab4f-981c27509a3a.png">

* `zlbytes`字段的类型是uint32_t, 这个字段中存储的是整个ziplist所占用的内存的字节数 
* `zltail`字段的类型是uint32_t, 它指的是ziplist中最后一个entry的偏移量. 用于快速定位最后一个entry, 以快速完成pop等操作
* `zllen`字段的类型是uint16_t, 它指的是整个ziplit中entry的数量. 这个值只占2bytes（16位）: 如果ziplist中entry的数目小于65535(2的16次方), 那么该字段中存储的就是实际entry的值. 若等于或超过65535, 那么该字段的值固定为65535, 但实际数量需要一个个entry的去遍历所有entry才能得到. 
* `zlend`是一个终止字节, 其值为全F, 即0xff. ziplist保证任何情况下, 一个entry的首字节都不会是255 ¶

### Entry

1. 字符串情况 -> `<prevlen> <encoding> <entry-data>`
 * `prevlen`：前一个entry的大小，
 * `encoding`：不同的情况下值不同，用于表示当前entry的类型和长度；
 * `entry-data`：真是用于存储entry表示的数据；

2. entry中存储的是int类型时， -> `<prevlen> <encoding>`
  * `encoding`和`entry-data`会合并在`encoding`中表示，此时没有entry-data字段
  * redis中，在存储数据时，会先尝试将string转换成int存储，节省空间；


**`prevlen`**
```
<prevlen from 0 to 253> <encoding> <entry>      //长度小于254结构,prevlen长度为1个字节
0xFE <4 bytes unsigned little endian prevlen> <encoding> <entry>   //长度大于等于254
// prevlen用5个字节表示，第一字节设置为254，后面4个字节存储一个小端的无符号整型，表示前一个entry的长度
```

**encoding编码**
* `encoding`的长度和值根据保存的是int还是string，还有数据的长度而定；
* 前两位用来表示类型，当为“11”时，表示entry存储的是`int`类型，其他表示存储的是string；

### 为什么ZipList特别省内存

* ziplist在设计时就很容易想到要尽量让每个元素按照实际的内容大小存储，所以增加encoding字段，针对不同的encoding来细化存储大小
 * 普通的数组，那么它每个元素占用的内存是一样的且取决于最大的那个元素
* 遍历元素时定位下一个元素呢？在普通数组中每个元素定长，所以不需要考虑这个问题；但是ziplist中每个data占据的内存不一样，所以为了解决遍历，需要增加记录上一个元素的length，所以增加了`prelen`字段

### ziplist的缺点
* ziplist也不预留内存空间, 并且在移除结点后, 也是立即缩容, 这代表每次写操作都会进行内存分配操作
* 结点如果扩容,导致结点占用的内存增长,并且超过254字节的话,可能会导致链式反应(其后一个结点的entry.prevlen需要从一字节扩容至五字节).最坏情况下,第一个结点的扩容,会导致整个ziplist表中的后续所有结点的entry.prevlen字段扩容.
----

## QuickList - 快表
> 一种以ziplist为结点的双端链表结构. 宏观上, quicklist是一个链表, 微观上, 链表中的每个结点都是一个ziplist。

<img width="606" alt="Screen Shot 2021-12-08 at 11 57 40 AM" src="https://user-images.githubusercontent.com/27160394/145145739-12775fe1-aaa4-46e0-9e87-8767bf03a51a.png">

* `quicklistNode`:这个结构描述的就是链表中的结点. 它通过zl字段持有底层的ziplist. 简单来讲, 它描述了一个ziplist实例
  * `quicklistNode.encoding`字段, 以指示本链表结点所持有的ziplist是否经过了压缩. 1代表未压缩, 持有的是原生的ziplist, 2代表压缩过
  * `quicklistNode.container`字段指示的是每个链表结点所持有的数据类型是什么
* `quicklistLZF`: ziplist是一段连续的内存, 用LZ4算法压缩后, 就可以包装成一个quicklistLZF结构. 是否压缩quicklist中的每个ziplist实例是一个可配置项. 若这个配置项是开启的, 那么quicklistNode.zl字段指向的就不是一个ziplist实例, 而是一个压缩后的quicklistLZF实例
* `quicklistBookmark`: 在quicklist尾部增加的一个书签，它只有在大量节点的多余内存使用量可以忽略不计的情况且确实需要分批迭代它们，才会被使用
* `quicklist`: 一个双链表的定义. 
  * `head,tail`分别指向头尾指针. len代表链表中的结点. 
  * `count`指的是整个quicklist中的所有ziplist中的entry的数目. 
  * `fill`字段影响着每个链表结点中ziplist的最大占用空间, 代表以字节数限制单个ziplist的最大长度
  * `compress`影响着是否zl字段指向的是原生的ziplist, 还是经过压缩包装后的quicklistLZF
* `quicklistEntry`: 对ziplist中的entry概念的封装. 
* `quicklistIter`: 一个迭代器
----
## Dict - 字典/哈希表
> 本质上就是哈希表, 哈希表是由数组 table 组成，table 中每个元素都是指向 dict.h/dictEntry 结构

<img width="543" alt="Screen Shot 2021-12-08 at 12 02 21 PM" src="https://user-images.githubusercontent.com/27160394/145146214-d5af3f60-2e65-474d-b896-012c39d62050.png">


> 如何解决哈希冲突

采用的便是链地址法，通过next这个指针可以将多个哈希值相同的键值对连接在一起，用来解决哈希冲突

> 扩容和收缩 : 当哈希表保存的键值对太多或者太少时，就要通过 rerehash(重新散列）来对哈希表进行相应的扩展或者收缩。

1. 如果执行扩展操作，会基于原哈希表创建一个大小等于`ht[0].used*2n`的哈希表（也就是每次扩展都是根据原哈希表已使用的空间扩大一倍创建另一个哈希表）。相反如果执行的是收缩操作，每次收缩是根据已使用空间缩小一倍创建一个新的哈希表。
2. 重新利用上面的哈希算法，计算索引值，然后将键值对放到新的哈希表位置上。
3. 所有键值对都迁徙完毕后，释放原哈希表的内存空间。

* 触发扩容的条件：

1. 服务器目前没有执行BGSAVE(生成RDB文件)命令或者 BGREWRITEAOF(用于异步执行一个AOF)命令，并且负载因子大于等于1。 
2. 服务器目前正在执行BGSAVE 命令或者 BGREWRITEAOF命令，并且负载因子大于等于5。
> 负载因子 = 哈希表已保存节点数量 / 哈希表大小。

> 渐近式 rehash
* 扩容和收缩操作不是一次性、集中式完成的，而是分多次、渐进式完成的
* 这样在进行渐进式rehash期间，字典的删除查找更新等操作可能会在两个哈希表上进行，第一个哈希表没有找到，就会去第二个哈希表上进行查找。
* 进行增加操作，一定是在新的哈希表上进行的

----
## IntSet - 整数集
> 当一个集合只包含整数值元素，并且这个集合的元素数量不多时，Redis 就会使用整数集合作为集合键的底层实现

```
typedef struct intset {
    uint32_t encoding; //表示编码方式，的取值有三个：INTSET_ENC_INT16, INTSET_ENC_INT32, INTSET_ENC_INT64
    uint32_t length; // 代表其中存储的整数的个数
    int8_t contents[]; // 指向实际存储数值的连续内存区域, 就是一个数组,整数集合的每个元素都是数组的一个item，各个项在数组中按值得大小从小到大有序排序，且数组中不包含任何重复项
} intset;
```

### 整数集合的升级
当在一个int16类型的整数集合中插入一个int32类型的值，整个集合的所有元素都会转换成32类型。 整个过程有三步：
1. 根据新元素的类型（比如int32），扩展整数集合底层数组的空间大小，并为新元素分配空间。
2. 将底层数组现有的所有元素都转换成与新元素相同的类型， 并将类型转换后的元素放置到正确的位上， 而且在放置元素的过程中， 需要继续维持底层数组的有序性质不变
3. 最后改变encoding的值，length+1。
* 如果我们删除掉刚加入的int32类型时，不会做一个降级操作, 主要还是减少开销的权衡
----
## ZSkipList - 跳表
> 作为有序列表 (Zset) 的使用

### 跳跃表
* 对于于一个单链表来讲，即便链表中存储的数据是有序的，如果我们要想在其中查找某个数据，也只能从头到尾遍历链表
* 增加层级降低查找复杂度

<img width="505" alt="Screen Shot 2021-12-08 at 12 14 23 PM" src="https://user-images.githubusercontent.com/27160394/145147139-f14d2d5c-f3e2-471d-b9f1-f27c7091ce6b.png">


### Redis跳跃表的设计

<img width="616" alt="Screen Shot 2021-12-08 at 12 34 54 PM" src="https://user-images.githubusercontent.com/27160394/145148800-6d59c2cf-408f-4e81-b4d3-8f44219355c1.png">


```
typedef struct zskiplistNode {
    robj *obj;
    double score;
    struct zskiplistNode *backward; //后向指针
    struct zskiplistLevel {
        struct zskiplistNode *forward;//每一层中的前向指针
        unsigned int span;//x.level[i].span 表示节点x在第i层到其下一个节点需跳过的节点数。注：两个相邻节点span为1
    } level[];
} zskiplistNode;

typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
```
* 头结点不持有任何数据, 且其`level[]`的长度为32 
* 每个结点
 * `redisObject` : 持有数据, robj 
 * `score`, 其标示着结点的得分, 结点之间凭借得分来判断先后顺序, 跳跃表中的结点按结点的得分升序排列. 
 * `backward`指针 ： 后向指针 该指针指向结点的前一个紧邻结点. 
 * `level`字段, 用以记录所有结点(除过头结点外)；每个结点中最多持有32个zskiplistLevel结构. 实际数量在结点创建时, 按幂次定律随机生成(不超过32). 每个zskiplistLevel中有两个字段
   * `forward`字段指向比自己得分高的某个结点(不一定是紧邻的), 并且, 若当前zskiplistLevel实例在level[]中的索引为X, 则其forward字段指向的结点, 其level[]字段的容量至少是X+1. 这也是上图中, 为什么forward指针总是画的水平的原因. 
   * `span`字段代表forward字段指向的结点, 距离当前结点的距离. 紧邻的两个结点之间的距离定义为1. ¶

### 什么不用平衡树或者哈希表

* They are not very memory intensive. It's up to you basically. Changing parameters about the probability of a node to have a given number of levels will make then less memory intensive than btrees.
* A sorted set is often target of many`ZRANGE`or`ZREVRANGE`operations, that is, traversing the skip list as a linked list. With this operation the cache locality of skip lists is at least as good as with other kind of balanced trees.
* They are simpler to implement, debug, and so forth. For instance thanks to the skip list simplicity I received a patch (already in Redis master) with augmented skip lists implementing ZRANK in O(log(N)). It required little changes to the code.

* skiplist和各种平衡树（如AVL、红黑树等）的元素是有序排列的，而哈希表不是有序的。因此，在哈希表上只能做单个key的查找，不适宜做范围查找。
* 在做范围查找的时候，平衡树比skiplist操作要复杂。在平衡树上，我们找到指定范围的小值之后，还需要以中序遍历的顺序继续寻找其它不超过大值的节点。
* 平衡树的插入和删除操作可能引发子树的调整，逻辑复杂，而skiplist的插入和删除只需要修改相邻节点的指针，操作简单又快速。
* 从算法实现难度上来比较，skiplist比平衡树要简单得多
