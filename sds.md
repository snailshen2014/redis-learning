SDS(simple dynamic string)是Redis提供的字符串封装，它兼容部分C字符串API，是许多其他数据结构的基础。

### 数据结构
redis提供了sdshdr5,sdshdr8,sdshdr16,sdshdr32,sdshdr64几种sdsd的实现，其中sdshdr5没有使用，
对应的源文件是sds.h,sds.c,我们以sdshdr8来举例，其定义如下：

```
struct sdshdr8{
  uint8_t len; /* used ，buf[]已使用大小 */
  uint8_t alloc; /* excluding the header and null terminator，buf[]分配的空间大小不包括字符串结尾\0 */
  unsigned char flags; /* 3 lsb of type, 5 unused bits ,标志位，表名sdshdr使用的类型*/
  char buf[]; /* 实际存储数据的字符串数组*/
}
```
sdshdr32 sdshdr64也和上面的结构一致，差别只在于len和alloc的数据类型不一样而已。相较于c原生的字符串，sds多了len、alloc、flag三个字段来存储一些额外的信息，redis考虑到了字符串拼接时带来的巨大损耗，所以每次新建sds的时候会预分配一些空间来应对未来的增长，正是因为预留空间的机制，所以redis需要记录下来已分配和总空间大小。
redis之所以分别提供了不通的sdshdr类型，是为了节约内存，尽量的少占用内存。

![sds](https://github.com/snailshen2014/redis-learning/blob/master/sds.jpg)


### 与C字符串的区别

| c字符串 | SDS |
| ------ | ------ |
| 获取字符串长度的复杂度O(N) | 获取字符串长度的复杂度为O(1) | 
| API是不安全的，可能会造成缓冲区溢出 |API是安全的，不会造成缓冲区溢出|
|修改字符串长度N次必然需要执行N次内存重分配|修改字符串长度N次最多需要执行N次重分配|
|只能保存文本|二进制安全可以保存任何数据|
|可以使用所有<string.h>的库函数|可以使用部分的C库函数|
