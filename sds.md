### 数据结构
```
struct sdshdr8{
  uint8_t len; /* used */
  uint8_t alloc; /* excluding the header and null terminator */
  unsigned char flags; /* 3 lsb of type, 5 unused bits */
  char buf[];
}
```

### 与C字符串的区别
| c字符串 | SDS |
| ------ | ------ |
| 获取字符串长度的复杂度O(N) | 获取字符串长度的复杂度为O(1) | 
| API是不安全的，可能会造成缓冲区溢出 |API是安全的，不会造成缓冲区溢出|
|修改字符串长度N次必然需要执行N次内存重分配|修改字符串长度N次最多需要执行N次重分配|
|只能保存文本|二进制安全可以保存任何数据|
|可以使用所有<string.h>的库函数|可以使用部分的C库函数|
