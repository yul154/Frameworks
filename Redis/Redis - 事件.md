# 事件机制
> Redis中的事件驱动库只关注网络IO,定时器。

Redis事件(`src/ae.c`)处理下面两类事件： 
* 文件事件(file event)：用于处理 Redis 服务器和客户端之间的网络IO
* 时间事件(time eveat)：Redis 服务器中的一些操作（比如serverCron函数）需要在给定的时间点执行，而时间事件就是处理这类定时操作的。

`aeEventLoop`是整个事件驱动的核心，它管理着文件事件表和时间事件列表，不断地循环处理着就绪的文件事件和到期的时间事件。

## 文件事件
> 基于Reactor模式开发了自己的网络事件处理器，也就是文件事件处理器

文件事件处理器使用IO多路复用技术,同时监听多个套接字，并为套接字关联不同的事件处理函数。当套接字的可读或者可写事件触发时，就会调用相应的事件处理函数。

### redis的IO多路复用
```
* Redis的瓶颈主要在IO而不是CPU
* Redis 是单线程主要是指Redis 的网络 IO 和键值对读写是由一个线程来完成的，这也是 Redis 对外提供键值存储服务的主要流程。
```

* Redis 使用的IO多路复用技术主要有：select、epoll、evport和kqueue等。每个IO多路复用函数库在 Redis 源码中都对应一个单独的文件，比如ae_select.c，ae_epoll.c， ae_kqueue.c

Redis事件响应框架`ae_event`及文件事件处理器
