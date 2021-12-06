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

sdshdr
* len 保存了SDS保存字符串的长度
*  buf[] 数组用来保存字符串的每个元素 
*  alloc分别以uint8, uint16, uint32, uint64表示整个SDS, 除过头部与末尾的\0, 剩余的字节数. 
*  flags 始终为一字节, 以低三位标示着头部的类型, 高5位未使用.

<img width="567" alt="Screen Shot 2021-12-06 at 6 51 14 PM" src="https://user-images.githubusercontent.com/27160394/144833530-d421367c-6fac-4265-aa73-da451811ff0a.png">


### SDS特点
* 常数复杂度获取字符串长度：获取 SDS 字符串的长度只需要读取 len 属性(`strlen key`)
* 杜绝缓冲区溢出: 在进行字符修改的时候，会首先根据记录的 len 属性检查内存空间是否满足需求，如果不满足，会进行相应的空间扩展(`strcat`函数来进行两个字符串的拼接)
* 减少修改字符串的内存重新分配次数:由于len属性和alloc属性的存在，对于修改字符串SDS实现了空间预分配和惰性空间释放
  * 空间预分配：对字符串进行空间扩展的时候，扩展的内存比实际需要的多，这样可以减少连续执行字符串增长操作所需的内存重分配次数。
  * 惰性空间释放：对字符串进行缩短操作时，程序不立即使用内存重新分配来回收缩短后多余的字节，而是使用 alloc 属性将这些字节的数量记录下来，等待后续使用

* 二进制安全:所有SDS的API都是以处理二进制的方式来处理`buf`里面的元素,并且 SDS 不是以空字符串来判断是否结束，而是以 len 属性表示的长度来判断字符串是否结束

> 空间预分配补进一步理解
