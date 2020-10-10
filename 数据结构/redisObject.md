### 对象的类型与编码
Redis 使用对象来表示数据库中的键和值， 每次当我们在 Redis 的数据库中新创建一个键值对时， 我们至少会创建两个对象， 一个对象用作键值对的键（键对象）， 另一个对象用作键值对的值（值对象）。
```
typedef struct redisObject
{
  unsigned type : 4;       //类型
  unsigned encoding : 4;   //编码
  unsigned lru : LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (least significant 8 bits frequency
                            * and most significant 16 bits access time). */
  int refcount;            //引用计数
  void *ptr;               //指向底层实现的数据结构
} robj;

```

### 类型
对象的 type 属性记录了对象的类型， 这个属性的值可以是下列出的常量的其中一个
|  类型常量   | 对象的名称  |
|  -----------  | --------  |
| REDIS_STRING  | 字符串对象 |
| REDIS_LIST  | 列表对象 |
| REDIS_HASH  | 哈希对象 |
| REDIS_SET  | 集合对象 |
| REDIS_ZSET  | 有序集合对象 |
对于 Redis 数据库保存的键值对来说， 键总是一个字符串对象， 而值则可以是字符串对象、列表对象、哈希对象、集合对象或者有序集合对象的其中一种， 因此：

* 当我们称呼一个数据库键为“字符串键”时， 我们指的是“这个数据库键所对应的值为字符串对象”；
* 当我们称呼一个键为“列表键”时， 我们指的是“这个数据库键所对应的值为列表对象”，
诸如此类。

TYPE 命令的实现方式也与此类似， 当我们对一个数据库键执行 TYPE 命令时， 命令返回的结果为数据库键对应的值对象的类型， 而不是键对象的类型

### 编码和底层实现
对象的 ptr 指针指向对象的底层实现数据结构， 而这些数据结构由对象的 encoding 属性决定。

encoding 属性记录了对象所使用的编码， 也即是说这个对象使用了什么数据结构作为对象的底层实现， 这个属性的值可以是表  列出的常量的其中一个。

|编码常量	|编码所对应的底层数据结构|
|--------|-------|
|REDIS_ENCODING_INT|	long 类型的整数|
|REDIS_ENCODING_EMBSTR|	embstr编码的简单动态字符串|
|REDIS_ENCODING_RAW	|简单动态字符串|
|REDIS_ENCODING_HT|	字典|
|REDIS_ENCODING_LINKEDLIST|	双端链表|
|REDIS_ENCODING_ZIPLIST|	压缩列表|
|REDIS_ENCODING_INTSET|	整数集合|
|REDIS_ENCODING_SKIPLIST|	跳跃表和字典|

每种类型的对象都至少使用了两种不同的编码， 表 列出了每种类型的对象可以使用的编码。

|类型	|编码	|对象|
|----|----|----|
|REDIS_STRING|	REDIS_ENCODING_INT|	使用整数值实现的字符串对象。|
|REDIS_STRING|	REDIS_ENCODING_EMBSTR|	使用 embstr 编码的简单动态字符串实现的字符串对象。|
|REDIS_STRING|	REDIS_ENCODING_RAW|	使用简单动态字符串实现的字符串对象。|
|REDIS_LIST|	REDIS_ENCODING_ZIPLIST|	使用压缩列表实现的列表对象。|
|REDIS_LIST	|REDIS_ENCODING_LINKEDLIST|	使用双端链表实现的列表对象。|
|REDIS_HASH|	REDIS_ENCODING_ZIPLIST|	使用压缩列表实现的哈希对象。|
|REDIS_HASH|	REDIS_ENCODING_HT|	使用字典实现的哈希对象。|
|REDIS_SET|	REDIS_ENCODING_INTSET|	使用整数集合实现的集合对象。|
|REDIS_SET|	REDIS_ENCODING_HT|	使用字典实现的集合对象。|
|REDIS_ZSET	|REDIS_ENCODING_ZIPLIST|	使用压缩列表实现的有序集合对象。|
|REDIS_ZSET	|REDIS_ENCODING_SKIPLIST|	使用跳跃表和字典实现的有序集合对象。|
使用 OBJECT ENCODING 命令可以查看一个数据库键的值对象的编码：
```
redis> SET msg "hello wrold"
OK

redis> OBJECT ENCODING msg
"embstr"

redis> SET story "long long long long long long ago ..."
OK

redis> OBJECT ENCODING story
"raw"

redis> SADD numbers 1 3 5
(integer) 3

redis> OBJECT ENCODING numbers
"intset"

redis> SADD numbers "seven"
(integer) 1

redis> OBJECT ENCODING numbers
"hashtable"
```

### 知识点
Redis 数据库中的每个键值对的键和值都是一个对象。

Redis 共有字符串、列表、哈希、集合、有序集合五种类型的对象， 每种类型的对象至少都有两种或以上的编码方式， 不同的编码可以在不同的使用场景上优化对象的使用效率。

服务器在执行某些命令之前， 会先检查给定键的类型能否执行指定的命令， 而检查一个键的类型就是检查键的值对象的类型。

Redis 的对象系统带有引用计数实现的内存回收机制， 当一个对象不再被使用时， 该对象所占用的内存就会被自动释放。

Redis 会共享值为 0 到 9999 的字符串对象。

对象会记录自己的最后一次被访问的时间， 这个时间可以用于计算对象的空转时间
