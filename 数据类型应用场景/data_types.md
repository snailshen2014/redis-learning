### 介绍
Redis的数据类型包括简单的键值数据类型String，集合类型（List,Set,Hash,Sorted sets)，Bit arrays,HyperLogLogs,Streams,Geo ，每种数据类型都有自己的使用场景，下文就依次说明每种数据类型的典型的应用场景。

### 应用场景

* String
    * 内部存储结构和编码方式  
    String类型内部分为三种编码方式(int,raw,embstr)  
        * 当你保存 64 位有符号整数时，String 类型会把它保存为一个 8 字节的 Long 类型整数，这种保存方式通常也叫作 int 编码方式。
        * 当你保存的数据中包含字符时，String 类型就会用简单动态字符串（Simple Dynamic String，SDS）结构体来保存，当保存的是字符串数据，并且字符串小于等于 44 字节时，RedisObject 中的元数据、指针和 SDS 是一块连续的内存区域，这样就可以避免内存碎片。这种布局方式也被称为 embstr 编码方式。当字符串大于 44 字节时，SDS 的数据量就开始变多了，Redis 就不再把 SDS 和 RedisObject 布局在一起了，而是会给 SDS 分配独立的空间，并用指针指向 SDS 结构。这种布局方式被称为 raw 编码模式。具体的编码结构如下图:  
        ![String](https://github.com/snailshen2014/redis-learning/blob/master/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF/String.jpg?raw=true)
    * 应用
    


* 集合类型（List,Set,Hash,Sorted sets)
* Bit arrays
* HyperLogLogs
* Streams
* Geo
 

