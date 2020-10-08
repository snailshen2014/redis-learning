整数集合（intset）是 Redis 用于保存整数值的集合抽象数据结构， 它可以保存类型为 int16_t 、 int32_t 或者 int64_t 的整数值， 并且保证集合中不会出现重复元素。

每个 intset.h/intset 结构表示一个整数集合：

```
typedef struct intset {

    // 编码方式
    uint32_t encoding;

    // 集合包含的元素数量
    uint32_t length;

    // 保存元素的数组
    int8_t contents[];

} intset;
```

contents 数组是整数集合的底层实现： 整数集合的每个元素都是 contents 数组的一个数组项（item）， 各个项在数组中按值的大小从小到大有序地排列， 并且数组中不包含任何重复项。

length 属性记录了整数集合包含的元素数量， 也即是 contents 数组的长度。

虽然 intset 结构将 contents 属性声明为 int8_t 类型的数组， 但实际上 contents 数组并不保存任何 int8_t 类型的值 —— contents 数组的真正类型取决于 encoding 属性的值：

如果 encoding 属性的值为 INTSET_ENC_INT16 ， 那么 contents 就是一个 int16_t 类型的数组， 数组里的每个项都是一个 int16_t 类型的整数值 （最小值为 -32,768 ，最大值为 32,767 ）。
如果 encoding 属性的值为 INTSET_ENC_INT32 ， 那么 contents 就是一个 int32_t 类型的数组， 数组里的每个项都是一个 int32_t 类型的整数值 （最小值为 -2,147,483,648 ，最大值为 2,147,483,647 ）。
如果 encoding 属性的值为 INTSET_ENC_INT64 ， 那么 contents 就是一个 int64_t 类型的数组， 数组里的每个项都是一个 int64_t 类型的整数值 （最小值为 -9,223,372,036,854,775,808 ，最大值为 9,223,372,036,854,775,807 ）。

图 6-1 展示了一个整数集合示例：

encoding 属性的值为 INTSET_ENC_INT16 ， 表示整数集合的底层实现为 int16_t 类型的数组， 而集合保存的都是 int16_t 类型的整数值。
length 属性的值为 5 ， 表示整数集合包含五个元素。
contents 数组按从小到大的顺序保存着集合中的五个元素。
因为每个集合元素都是 int16_t 类型的整数值， 所以 contents 数组的大小等于 sizeof(int16_t) * 5 = 16 * 5 = 80 位。

![intset1](https://github.com/snailshen2014/redis-learning/blob/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/intset1.png)


图 6-2 展示了另一个整数集合示例：

encoding 属性的值为 INTSET_ENC_INT64 ， 表示整数集合的底层实现为 int64_t 类型的数组， 而数组中保存的都是 int64_t 类型的整数值。
length 属性的值为 4 ， 表示整数集合包含四个元素。
contents 数组按从小到大的顺序保存着集合中的四个元素。
因为每个集合元素都是 int64_t 类型的整数值， 所以 contents 数组的大小为 sizeof(int64_t) * 4 = 64 * 4 = 256 位
![intset1](https://github.com/snailshen2014/redis-learning/blob/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/intset2.png)
