### 介绍
Redis的数据类型包括简单的键值数据类型String，集合类型（List,Set,Hash,Sorted sets)，Bit arrays,HyperLogLogs,Streams,Geo ，每种数据类型都有自己的使用场景，下文就依次说明每种数据类型的典型的应用场景。

### 应用场景

* String
 String类型内部分为三种编码方式，当你保存 64 位有符号整数时，String 类型会把它保存为一个 8 字节的 Long 类型整数，这种保存方式通常也叫作 int 编码方式。但是，当你保存的数据中包含字符时，String 类型就会用简单动态字符串（Simple Dynamic String，SDS）结构体来保存，结构如下：len(4B)|alloc(4B)|buf("redis\0")
buf：字节数组，保存实际数据。为了表示字节数组的结束，Redis 会自动在数组最后加一个“\0”，这就会额外占用 1 个字节的开销。len：占 4 个字节，表示 buf 的已用长度。alloc：也占个 4 字节，表示 buf 的实际分配长度，一般大于 len。可以看到，在 SDS 中，buf 保存实际数据，而 len 和 alloc 本身其实是 SDS 结构体的额外开销。因为 Redis 的数据类型有很多，而且，不同数据类型都有些相同的元数据要记录（比如最后一次访问的时间、被引用的次数等），所以，Redis 会用一个 RedisObject 结构体来统一记录这些元数据，同时指向实际数据。一个 RedisObject 包含了 8 字节的元数据和一个 8 字节指针，这个指针再进一步指向具体数据类型的实际数据所在，例如指向 String 类型的 SDS 结构所在的内存地址，可以看一下下面的示意图。




* 集合类型（List,Set,Hash,Sorted sets)
* Bit arrays
* HyperLogLogs
* Streams
* Geo
 

