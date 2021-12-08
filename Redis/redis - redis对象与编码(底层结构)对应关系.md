**Redis的每种对象其实都由对象结构(redisObject) 与 对应编码的数据结构组合而成,**

<img width="573" alt="Screen Shot 2021-12-06 at 6 03 52 PM" src="https://user-images.githubusercontent.com/27160394/144826875-74ed825f-cb13-4de9-97ff-7d2c4ddab442.png">


# 字符串对象
字符串对象的编码可以是int，raw或者embstr
* int 编码：保存的是可以用 long 类型表示的整数值。
* embstr 编码：保存长度小于44字节的字符串（redis3.2版本之前是39字节，之后是44字节）
* raw 编码：保存长度大于44字节的字符串（redis3.2版本之前是39字节，之后是44字节）

> int 编码是用来保存整数值，而embstr是用来保存短字符串，raw编码是用来保存长字符串。Redis中对于浮点数类型也是作为字符串保存的，在需要的时候再将其转换成浮点数类型

**raw 和 embstr 的区别**
* embstr与raw都使用redisObject和sds保存数据，
  * embstr的使用只分配一次内存空间（因此redisObject和sds是连续的），embstr的好处在于创建时少分配一次空间，删除时少释放一次空间，以及对象的所有数据连在一起，寻找方便
      * 如果字符串的长度增加需要重新分配内存时，整个redisObject和sds都需要重新分配空间，因此redis中的embstr实现为只读
  * 而raw需要分配两次内存空间（分别为redisObject和sds分配空间）


编码的转换
* Redis中对于浮点数类型也是作为字符串保存的，在需要的时候再将其转换成浮点数类型
* 当 int 编码保存的值不再是整数，或大小超过了long的范围时，自动转化为raw。
* 由于embstr是只读的，在对embstr对象进行修改时，都会先转化为raw再进行修改，因此，只要是修改embstr对象，修改后的对象一定是raw的，无论是否达到了44个字节
-----

# 列表对象 List
> 按照插入顺序排序,添加一个元素到列表的头部(左边)或者尾部(右边)，它的底层实际上是个链表结构。

列表对象的编码是quicklist

<img width="583" alt="Screen Shot 2021-12-08 at 12 50 55 PM" src="https://user-images.githubusercontent.com/27160394/145150178-07543f82-9153-4679-8ba7-0c46d2ed9f2f.png">

----

# HashMap - 哈希对象

哈希对象的编码可以是 ziplist 或者 hashtable；对应的底层实现有两种, 一种是ziplist, 一种是dict。

<img width="612" alt="Screen Shot 2021-12-08 at 12 53 29 PM" src="https://user-images.githubusercontent.com/27160394/145150351-2289aa7c-291d-45f6-9e22-ffa6332f91e6.png">

* 当使用ziplist，也就是压缩列表作为底层实现时，新增的键值对是保存到压缩列表的表尾
* hashtable 编码的哈希表对象底层使用字典数据结构，哈希对象中的每个键值对都使用一个字典键值对

## 编码转换
编码类型转换在Redis写入数据时自动完成，这个转换过程是不可逆的，转换规则只能从小内存编码向大内存编码转换

当同时满足下面两个条件时，使用ziplist（压缩列表）编码：
1. 列表保存元素个数小于512个
2. 每个元素长度小于64字节
3. 不能满足这两个条件的时候使用 hashtable 编码

----
# Set - 集合对象

集合对象的编码
* intset - `intset`(集合中存储的只能是数值数据, 且必须是整数)
* hashtable - `dict`(将数据全部存储于dict的键中, 值字段闲置不用)

<img width="503" alt="Screen Shot 2021-12-08 at 1 00 35 PM" src="https://user-images.githubusercontent.com/27160394/145150976-40ad180c-90ac-4f18-9fd2-e8a02e802aec.png">

## 编码转换

当集合同时满足以下两个条件时，使用 intset 编码： 
1. 集合对象中所有元素都是整数
2. 集合对象所有元素数量不超过512 不能满足这两个条件的就使用 hashtable 编码
3. 第二个条件可以通过配置文件的`set-max-intset-entries`进行配置。

----
# Zset - 有序集合对象

有序集合的底层实现依然有两种,
* 使用ziplist作为底层实现  - ZIPLIST
* 底层使用了两种数据结构: dict与skiplist.  - SKIPLIST

<img width="343" alt="Screen Shot 2021-12-08 at 1 04 52 PM" src="https://user-images.githubusercontent.com/27160394/145151349-ef5c5562-b905-4600-8e43-51d0dd2791c4.png">



* 假如我们单独使用 字典，虽然能以 O(1) 的时间复杂度查找成员的分值，但是因为字典是以无序的方式来保存集合元素，所以每次进行范围操作的时候都要进行排序；
* 假如我们单独使用跳跃表来实现，虽然能执行范围操作，但是查找操作有 O(1)的复杂度变为了O(logN)。
* 因此Redis使用了两种数据结构来共同实现有序集合。

**编码转换**
当有序集合对象同时满足以下两个条件时，对象使用 ziplist 编码：

1. 保存的元素数量小于128；
2. 保存的所有元素长度都小于64字节。
3. 
不能满足上面两个条件的使用 skiplist 编码
