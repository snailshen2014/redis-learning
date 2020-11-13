### 介绍
Redis的数据类型包括简单的键值数据类型String，集合类型（List,Set,Hash,Sorted sets)，Bit arrays,HyperLogLogs,Streams,Geo ，每种数据类型都有自己的使用场景，下文就依次说明每种数据类型的典型的应用场景。

### 数据类型

* String
    * 内部存储结构和编码方式  
    String类型内部分为三种编码方式(int,raw,embstr)  
        * 当你保存 64 位有符号整数时，String 类型会把它保存为一个 8 字节的 Long 类型整数，这种保存方式通常也叫作 int 编码方式。
        * 当你保存的数据中包含字符时，String 类型就会用简单动态字符串（Simple Dynamic String，SDS）结构体来保存，当保存的是字符串数据，并且字符串小于等于 44 字节时，RedisObject 中的元数据、指针和 SDS 是一块连续的内存区域，这样就可以避免内存碎片。这种布局方式也被称为 embstr 编码方式。当字符串大于 44 字节时，SDS 的数据量就开始变多了，Redis 就不再把 SDS 和 RedisObject 布局在一起了，而是会给 SDS 分配独立的空间，并用指针指向 SDS 结构。这种布局方式被称为 raw 编码模式。具体的编码结构如下图:  
        ![String](https://github.com/snailshen2014/redis-learning/blob/master/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF/String.jpg?raw=true)  
        
    * 应用场景
        * 分布式锁
        * 计数器
        * K/V存储

* 集合类型（List,Set,Hash,Sorted sets)
    * 聚合统计  
    所谓的聚合统计，就是指统计多个集合元素的聚合结果，包括：统计多个集合的共有元素（交集统计）；把两个集合相比，统计其中一个集合独有的元素（差集统计）；统计多个集合的所有元素（并集统计）。
        * 应用场景
            * 在移动应用中，需要统计每天的新增用户数和第二天的留存用户数  
              实现步骤通过设定两个set(A,B)做差集来实现，A叫user-id保存的是所有的登录的用户id,B user-id-20201113日登录的用户id,那么  
              SDIFFSTORE  user-new  user-id-20201113 user-id 运算后计算出来的user:new就是2020年11月13日留存的用户数量（前提是user:id记录了用户的id)  
              注意事项：Set 的差集、并集和交集的计算复杂度较高，在数据量较大的情况下，如果直接执行这些计算，会导致 Redis 实例阻塞。所以，我给你分享一个小建议：你可以从主从集群中选择一个  
              从库，让它专门负责聚合计算，或者是把数据读取到客户端，在客户端来完成聚合统计，这样就可以规避阻塞主库实例和其他从库实例的风险了。  
     
    * 排序统计  
     我以在电商网站上提供最新评论列表的场景为例，进行讲解。最新评论列表包含了所有评论中的最新留言，这就要求集合类型能对元素保序，也就是说，集合中的元素可以按序排列，这种对元素保序的集合类型叫作有      序集合。在 Redis 常用的 4 个集合类型中（List、Hash、Set、Sorted Set），List 和 Sorted Set 就属于有序集合。List 是按照元素进入 List 的顺序进行排序的，而 Sorted Set 可以根据元素      的权重来排序，我们可以自己来决定每个元素的权重值。比如说，我们可以根据元素插入 Sorted Set 的时间确定权重值，先插入的元素权重小，后插入的元素权重大。
        * 应用场景  
            * 商品评论，博客评论  
            通过list的lrange,或者通过Sorted sets的ZRANGEBYSCORE
        

            
    
    * 排序统计
    * 二值状态统计
    * 基数统计

* Bit arrays
* HyperLogLogs
* Streams
* Geo
 

