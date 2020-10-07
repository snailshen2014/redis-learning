链表在Redis中的应用非常广泛，比如列表的底层实现之一就是链表。当一个列表的键包含了数量比较多的元素，又或者列表中包含的元素都是比较长的字符串时，Redis就会使用链表做为列表键的底层实现。
除了列表之外，发布与订阅、慢查询、监视器等功能也用到了链表。

### 数据结构

```
typedef struct listNode
{
  struct listNode *prev; /*前置节点指针*/
  struct listNode *next; /*后置节点指针*/
  void *value;           /*节点的值*/
} listNode;

typedef struct listIter
{
  listNode *next;
  int direction;
} listIter;

typedef struct list
{
  listNode *head;                     /*表头节点*/
  listNode *tail;                     /*表尾节点*/
  void *(*dup)(void *ptr);            /*节点复制函数*/
  void (*free)(void *ptr);            /*节点释放函数*/
  int (*match)(void *ptr, void *key); /*节点值对比函数*/
  unsigned long len;                  /*节点长度*/
} list;
```



