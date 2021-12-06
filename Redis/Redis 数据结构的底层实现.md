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

`sdshdr`
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

著作权归https://pdai.tech所有。

当执行追加操作时，比如现在给key=‘Hello World’的字符串后追加‘ again!’则这时的len=18，free由0变成了18，
* 此时的buf='Hello World again!\0....................'(.表示空格)，也就是buf的内存空间是18+18+1=37个字节，其中‘\0’占1个字节redis给字符串多分配了18个字节的预分配空间，
* 所以下次还有append追加的时候，如果预分配空间足够，就无须在进行空间分配了。
* 在当前版本中，当新字符串的长度小于1M时，redis会分配他们所需大小一倍的空间，当大于1M的时候，就为他们额外多分配1M的空间。

----
## 压缩列表 - ZipList

ziplist是为了提高存储效率而设计的一种特殊编码的双向链表。
* 它可以存储字符串或者整数
* 存储整数时是采用整数的二进制而不是字符串形式存储。
* 他能在O(1)的时间复杂度下完成list两端的push和pop操作。但是因为每次操作都需要重新分配ziplist的内存，所以实际复杂度和ziplist的内存使用量相关。

<img width="463" alt="Screen Shot 2021-12-06 at 10 40 23 PM" src="https://user-images.githubusercontent.com/27160394/144865629-c46bbc9d-73f0-4d8b-ab4f-981c27509a3a.png">
著作权归https://pdai.tech所有。
链接：https://pdai.tech/md/db/nosql-redis/db-redis-x-redis-ds.html

* `zlbytes`字段的类型是uint32_t, 这个字段中存储的是整个ziplist所占用的内存的字节数 
* `zltail`字段的类型是uint32_t, 它指的是ziplist中最后一个entry的偏移量. 用于快速定位最后一个entry, 以快速完成pop等操作
* `zllen`字段的类型是uint16_t, 它指的是整个ziplit中entry的数量. 这个值只占2bytes（16位）: 如果ziplist中entry的数目小于65535(2的16次方), 那么该字段中存储的就是实际entry的值. 若等于或超过65535, 那么该字段的值固定为65535, 但实际数量需要一个个entry的去遍历所有entry才能得到. 
* `zlend`是一个终止字节, 其值为全F, 即0xff. ziplist保证任何情况下, 一个entry的首字节都不会是255 ¶
