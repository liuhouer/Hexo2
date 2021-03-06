title: "如何简单地比较Redis与Memcached的区别"
date: 2015-03-16 10:04:45
tags: [数据缓存,java]
categories: 缓存
---

如果简单地比较Redis与Memcached的区别，大多数都会得到以下观点：

- Redis不仅仅支持简单的key/value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。

- Redis支持数据的备份，即master-slave模式的数据备份。

- Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用+数据同步
      

<!--more-->


这两年 Redis火得可以，Redis也常常被当作 Memcached的挑战者被提到桌面上来。关于Redis与Memcached的比较更是比比皆是。然而，Redis真的在功能、性能以及内存使用效率上都超越了Memcached吗？
没有必要过于关注性能，因为二者的性能都已经足够高了。由于Redis只使用单核，而Memcached可以使用多核，所以二者比较起来，平均每一个核上，Redis在存储小数据时比Memcached性能更高。而在100k以上的数据中，Memcached性能要高于Redis。虽然Redis最近也在存储大数据的性能上进行优化，但是比起Memcached，还是稍有逊色。说了这么多，结论是，无论你使用哪一个，每秒处理请求的次数都不会成为瓶颈。
在内存使用效率上，如果使用简单的key-value存储，Memcached的内存利用率更高。而如果Redis采用hash结构来做key-value存储，由于其组合式的压缩，其内存利用率会高于Memcached。当然，这和你的应用场景和数据特性有关。
如果你对数据持久化和数据同步有所要求，那么推荐你选择Redis。因为这两个特性Memcached都不具备。即使你只是希望在升级或者重启系统后缓存数据不会丢失，选择Redis也是明智的。
当然，最后还得说到你的具体应用需求。Redis相比Memcached来说，拥有更多的数据结构，并支持更丰富的数据操作。通常在Memcached里，你需要将数据拿到客户端来进行类似的修改再set回去。这大大增加了网络IO的次数和数据体积。在Redis中，这些复杂的操作通常和一般的GET/SET一样高效。所以，如果你需要缓存能够支持更复杂的结构和操作，那么Redis会是不错的选择。


