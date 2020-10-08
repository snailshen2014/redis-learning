### layout
```
<zlbytes> <zltail> <zllen> <entry> <entry> ... <entry> <zlend>
```

详细说明

属性	类型	长度	用途

zlbytes	uint32_t	4 字节	记录整个压缩列表占用的内存字节数：在对压缩列表进行内存重分配， 或者计算 zlend 的位置时使用。

zltail	uint32_t	4 字节	记录压缩列表表尾节点距离压缩列表的起始地址有多少字节： 通过这个偏移量，程序无须遍历整个压缩列表就可以确定表尾节点的地址。

zllen	uint16_t	2 字节	记录了压缩列表包含的节点数量： 当这个属性的值小于 UINT16_MAX （65535）时， 这个属性的值就是压缩列表包含节点的数量； 当这个值等于 UINT16_MAX 时， 节点的真实数量需要遍历整个压缩列表才能计算得出。

entryX	列表节点	不定	压缩列表包含的各个节点，节点的长度由节点保存的内容决定。

zlend	uint8_t	1 字节	特殊值 0xFF （十进制 255 ），用于标记压缩列表的末端。

举个例子

![ziplist](https://github.com/snailshen2014/redis-learning/blob/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/ziplist.jpg)
