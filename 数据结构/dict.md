Redis 的字典使用哈希表作为底层实现， 一个哈希表里面可以有多个哈希表节点， 而每个哈希表节点就保存了字典中的一个键值对。

接下来的三个小节将分别介绍 Redis 的哈希表、哈希表节点、以及字典的实现。

### 哈希表
Redis 字典所使用的哈希表由 dict.h/dictht 结构定义
```
typedef struct dictht
{
  dictEntry **table;      //哈希数组
  unsigned long size;     //哈希表大小 /* This is the initial size of every hash table */ #define DICT_HT_INITIAL_SIZE     4 ,默认大小为4，会动态扩展
  unsigned long sizemask; //哈希表掩码用于计算索引值，总是等于size-1
  unsigned long used;     //哈希表已有节点的数量
} dictht;

```
table 属性是一个数组， 数组中的每个元素都是一个指向 dict.h/dictEntry 结构的指针， 每个 dictEntry 结构保存着一个键值对。

size 属性记录了哈希表的大小， 也即是 table 数组的大小， 而 used 属性则记录了哈希表目前已有节点（键值对）的数量。

sizemask 属性的值总是等于 size - 1 ， 这个属性和哈希值一起决定一个键应该被放到 table 数组的哪个索引上面。

### 哈希表节点
哈希表节点使用 dictEntry 结构表示， 每个 dictEntry 结构都保存着一个键值对：
```
typedef struct dictEntry
{
  void *key; //键
  //值
  union
  {
    void *val;
    uint64_t u64;
    int64_t s64;
    double d;
  } v;
  //指向下一个哈希表节点，形成链表
  struct dictEntry *next;
} dictEntry;
```

key 属性保存着键值对中的键， 而 v 属性则保存着键值对中的值， 其中键值对的值可以是一个指针， 或者是一个 uint64_t 整数， 又或者是一个 int64_t 整数。

next 属性是指向另一个哈希表节点的指针， 这个指针可以将多个哈希值相同的键值对连接在一次， 以此来解决键冲突（collision）的问题。

### 字典

Redis 中的字典由 dict.h/dict 结构表示：
```
typedef struct dict
{
  dictType *type;
  void *privdata;          //私有数据
  dictht ht[2];            //哈希表
  long rehashidx;          /* rehashing not in progress if rehashidx == -1 */
  unsigned long iterators; /* number of iterators currently running */
} dict;
```

type 属性和 privdata 属性是针对不同类型的键值对， 为创建多态字典而设置的：

type 属性是一个指向 dictType 结构的指针， 每个 dictType 结构保存了一簇用于操作特定类型键值对的函数， Redis 会为用途不同的字典设置不同的类型特定函数。
而 privdata 属性则保存了需要传给那些类型特定函数的可选参数

```
//不同类型不通操作
typedef struct dictType
{
  uint64_t (*hashFunction)(const void *key);
  void *(*keyDup)(void *privdata, const void *key);
  void *(*valDup)(void *privdata, const void *obj);
  int (*keyCompare)(void *privdata, const void *key1, const void *key2);
  void (*keyDestructor)(void *privdata, void *key);
  void (*valDestructor)(void *privdata, void *obj);
} dictType;
```

ht 属性是一个包含两个项的数组， 数组中的每个项都是一个 dictht 哈希表， 一般情况下， 字典只使用 ht[0] 哈希表， ht[1] 哈希表只会在对 ht[0] 哈希表进行 rehash 时使用。

除了 ht[1] 之外， 另一个和 rehash 有关的属性就是 rehashidx ： 它记录了 rehash 目前的进度， 如果目前没有在进行 rehash ， 那么它的值为 -1 。


### 重点知识点
**字典被广泛用于实现Redis的各种功能，其中包括数据库和哈希键**

**Redis中的字典使用哈希表做为底层实现，每个字典有两个哈希表，一个平时使用，另外一个在rehash使用**

**当字典被用作数据库的底层实现，或者哈希键的底层实现时，Rediss使用MurmurHash2算法那来计算键的哈希值**

**哈希表使用链表来解决哈希冲突**

**渐进式rehash,哈希因子**
https://luoming1224.github.io/2018/11/12/[redis%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0]redis%E6%B8%90%E8%BF%9B%E5%BC%8Frehash%E6%9C%BA%E5%88%B6/
