# Redis发布订阅

Redis的`SUBSCRIBE`命令可以让客户端订阅任意数量的频道， 每当有新信息发送到被订阅的频道时， 信息就会被发送给所有订阅指定频道的客户端。

Redis有两种发布/订阅模式：
* 基于频道(Channel)的发布/订阅
* 基于模式(pattern)的发布/订阅

## 基于频道(Channel)的发布/订阅
> 发布者可以向指定的频道(channel)发送消息; 订阅者可以订阅一个或者多个频道(channel)

发布者发布消息的命令是 publish,用法是 `publish channel message`
```
publish channel:1 hi //返回值表示接收这条消息的订阅者数量,消息不会被持久化
```
订阅频道的命令是 subscribe，可以同时订阅多个频道，用法是 `subscribe channel1 [channel2 ...]`
```
127.0.0.1:6379> subscribe channel:1  //执行上面命令客户端会进入订阅状态
Reading messages... (press Ctrl-C to quit) // 处于此状态下客户端不能使用除subscribe、unsubscribe、psubscribe和punsubscribe这四个属于"发布/订阅"之外的命令
1) "subscribe" // 消息类型
2) "channel:1" // 频道
3) "hi" // 消息内容
```

### 基于频道(Channel)的发布/订阅如何实现的？

底层是通过字典实现的
* 这个字典就用于保存订阅频道的信息：字典的键为正在被订阅的频道，而字典的值则是一个链表，链表中保存了所有订阅这个频道的客户端。

<img width="551" alt="Screen Shot 2021-12-07 at 8 42 39 PM" src="https://user-images.githubusercontent.com/27160394/145031086-507ca490-c3b0-4602-9eb2-5fb52b1f8dd8.png">

* 当客户端调用`SUBSCRIBE`命令时， 程序就将客户端和要订阅的频道在 pubsub_channels 字典中关联起来
* 当调用`PUBLISH channel message`命令， 程序首先根据`channel`定位到字典的键， 然后将信息发送给字典值链表中的所有客户端。
* 使用`UNSUBSCRIBE`命令可以退订指定的频道， 这个命令执行的是订阅的反操作： 它从 pubsub_channels 字典的给定频道（键）中， 删除关于当前客户端的信息


## 基于模式(pattern)的发布/订阅
如果有某个/某些模式和这个频道匹配的话，那么所有订阅这个/这些频道的客户端也同样会收到信息

<img width="528" alt="Screen Shot 2021-12-07 at 8 37 44 PM" src="https://user-images.githubusercontent.com/27160394/145030390-13cc738f-5058-47b6-bae4-97ab87413072.png">

* 使用`psubscribe`命令可以重复订阅同一个频道，如客户端执行了`psubscribe c? c?*`。这时向c1发布消息客户端会接受到两条消息，而同时publish命令的返回值是2而不是1
* `punsubscribe`命令可以退订指定的规则，用法是: `punsubscribe [pattern [pattern ...]]`,如果没有参数则会退订所有规则。
* 使用`punsubscribe`只能退订通过`psubscribe`命令订阅的规则，不会影响直接通过`subscribe`命令订阅的频道；同样`unsubscribe`命令也不会影响通过`psubscribe`命令订阅的规则
* `punsubscribe`命令退订某个规则时不会将其中的通配符展开，而是进行严格的字符串匹配，所以`punsubscribe *` 无法退订c* 规则，而是必须使用`punsubscribe c*`才可以退订

### 基于模式(Pattern)的发布/订阅如何实现的？
> 底层是pubsubPattern节点的链表。
*  每当调用`PSUBSCRIBE`命令订阅一个模式时， 程序就创建一个包含客户端信息和被订阅模式的 pubsubPattern 结构， 并将该结构添加到 redisServer.pubsub_patterns 链表中
