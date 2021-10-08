> 包含书本第一部分 2-8 章节的内容。

Redis 是一个对象系统，每个对象拥有以下基本属性：

* 类型 type

  目前总共有五种对象类型。最直观的作用是用于判定一个对象是否可以执行给定的 Redis 命令。

  当我们在 Redis 创建一个键值对时，Redis 至少会创建两个对象：键对象与值对象。

* 编码 encoding

  目前共有八种编码方式。

  编码方式决定一个类型底层实现所用到的数据结构，同一个类型可能有不止一种编码方式。

  Redis 针对不同的使用场景，为同一个类型设置多种底层数据结构实现，优化使用效率。

* 引用计数 refcount

  对象的内存回收机制。引用计数归零时，对象就会被回收。

  除此之外，还可以用于对象共享（Redis 会共享 0 -9999 的字符串对象）以节约内存。

* 对象最后一次访问时间 lru

  记录对象最近一次的访问时间，用于计算空转时长。如果开启了了 max-memory，空转时长大的对象会优先被数据库删除。

* 指向实际值的指针

  真正的数据存储的地址。

本文组织：首先介绍对象的类型与编码之间的羁绊，然后介绍对象的内存机制（对应书本 8 章节），最后给出编码方式背后的数据结构（对应书本 2-7 章节）。

# 1 - 对象的类型与编码

## 1.1 类型与编码方式的映射

**五种对象类型**

| 对象         | type 属性的值  | `TYPE` 命令的输出 | REDIS 命令示例               |
| ------------ | -------------- | ----------------- | ---------------------------- |
| 字符串对象   | `REDIS_STRING` | "string"          | *SET、GET、APPEND、STRLEN*   |
| 列表对象     | `REDIS_LIST`   | "list"            | *RPUSH、LPOP、LINSERT、LLEN* |
| 哈希对象     | `REDIS_HASH`   | "hash"            | *HSET、HGET、HLEN、HDEL*     |
| 集合对象     | `REDIS_SET`    | "set"             | *SADD、SPOP、SINTER、SCARD*  |
| 有序集合对象 | `REDIS_ZSET`   | "zset"            | *ZADD、ZCARD、ZRANK、ZSCORE* |

**八种编码方式**

* ”列表键“指的是指向列表对象的键，同理“字符串键”。
* 列表发布和订阅：`RPUSH` + `LPOP`

| 编码常量                    | 对应底层数据结构  | `OBJECT ENCODING `输出 | 复杂度                              | 用途                                 |
| --------------------------- | ----------------- | ---------------------- | ----------------------------------- | ------------------------------------ |
| `REDIS_ENCODING_INT`        | long 类型的整数   | "int"                  |                                     |                                      |
| `REDIS_ENCODING_EMBSTR`     | embstr 编码的 SDS | "embstr"               |                                     |                                      |
| `REDIS_ENCODING_RAW`        | SDS               | "raw"                  |                                     | 字符串键、用作缓冲区（buffer)        |
| `REDIS_ENCODING_HT`         | 字典              | "hashtable"            | O(1)                                | 字典键、Redis 数据库                 |
| `REDIS_ENCODING_LINKEDLIST` | 双端链表          | "linkedlist"           | 查找或者 给定 index O(N)            | 列表键、发布和订阅、慢查询和监视器等 |
| `REDIS_ENCODING_ZIPLIST`    | 压缩列表          | "ziplist"              | N                                   | 列表键和哈希键的底层实现             |
| `REDIS_ENCODING_INTSET`     | 整数集合          | "intset"               | 查找、插入：N                       | 集合键底层实现（只适用于整数集合）   |
| `REDIS_ENCODING_SKIPLIST`   | 跳跃表和字典      | "skiplist"             | 插入、删除、查找：平均 logN，最坏 N | 有序集合底层实现                     |

**类型和编码的映射关系**

* 一个对象实例的编码方式并不是一成不变的，随着长度的增加，字符串对象实例的编码方式可能从 embstr 变为 SDS。
* 编码转化的阈值可以手动设置。
* ziplist  三杀，用于列表对象、哈希对象和有序集合对象。

| 类型           | 编码                        | 对象                                    | 使用场景                                                     |
| -------------- | --------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| `REDIS_STRING` | `REDIS_ENCODING_INT`        | 使用整数值实现的字符串对象              | 保存的是一个整数值，并且这个整数值能够用 long 类型表示；     |
| `REDIS_STRING` | `REDIS_ENCODING_EMBSTR`     | 使用 embstr 编码的 SDS 实现的字符串对象 | 保存的字符串值的 size 小于等于 32 字节；                     |
| `REDIS_STRING` | `REDIS_ENCODING_RAW`        | 使用 SDS 实现的字符串对象               | 保存的字符串值的 size 大于 32 字节；                         |
|                |                             |                                         |                                                              |
| `REDIS_LIST`   | `REDIS_ENCODING_ZIPLIST`    | 使用压缩列表实现的列表对象              | 列表对象保存的元素同时满足1）size 都小于 64 字节且2）元素数量小于 512 个； |
| `REDIS_LIST`   | `REDIS_ENCODING_LINKEDLIST` | 使用双端链表实现的列表对象              | 上述条件不满足则将选择（转化为）双端链表实现；               |
|                |                             |                                         |                                                              |
| `REDIS_HASH`   | `REDIS_ENCODING_ZIPLIST`    | 使用压缩列表实现的哈希对象              | 同时满足1）所有键值对的键和值的 size  小于 64 字节且2）键值对的数量小于512； |
| `REDIS_HASH`   | `REDIS_ENCODING_HT`         | 使用字典实现的哈希对象                  | 上述条件不满足时，使用 hashtable 实现；                      |
|                |                             |                                         |                                                              |
| `REDIS_SET`    | `REDIS_ENCODING_INTSET`     | 使用整数集合实现的集合对象              | 同时满足 1）元素都是整数值且 2）数量不超过 512 个；          |
| `REDIS_SET`    | `REDIS_ENCODING_HT`         | 使用字典实现的集合对象                  | 上述条件不满足，使用 hashtable 实现；                        |
|                |                             |                                         |                                                              |
| `REDIS_ZSET`   | `REDIS_ENCODING_ZIPLIST`    | 使用压缩列表实现的有序集合对象          | 同时满足1）元素 size 小于 64 字节且2）元素数量小于128个；    |
| `REDIS_ZSET`   | `REDIS_ENCODING_SKIPLIST`   | 使用跳跃表实现的有序集合对象            | 上述条件不满足，使用跳跃表实现；                             |

## 1.2 五种对象类型

直接参考[RedisBook 图集](http://1e-gallery.redisbook.com/8-object.html#id2) 会更清晰一点。

### 1.2.1 字符串对象

* embstr v.s. SDS

  embstr 是针对短字符串进行优化的一种编码方式。

  二者都拥有 redisObject 和 sdshdr 两种结构，区别在于调用内存分配函数的次数。

  embstr 调用一次，为 redisObject 和 sdshdr 申请一块连续的空间，回收内存时也只需调用一次内存释放函数。

  SDS 调用内存分配函数两次，回收时 也需要调用两次内存释放函数。

  优势在于：1）**连续的内存能够更好的利用缓存优势**；2）**系统函数（内存分配/释放函数）被调用次数减少**。

<figure>
  <img src=Redis设计与实现.assets/image-20210929105700647-2884224.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-1 使用整数值实现的字符串对象。</figcaption>
  <img src=Redis设计与实现.assets/image-20210929105739510-2884286.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-2 使用 embstr 实现的字符串对象。</figcaption>
  <img src=Redis设计与实现.assets/image-20210929105721489.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-3 使用 SDS 实现的字符串对象。</figcaption>
</figure>


### 1.2.2 列表对象

执行 `RPUSH mylist 1 "three" 5`，两种实现方式对比。

<figure>
  <img src=Redis设计与实现.assets/image-20210929111802590.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-4 使用压缩列表实现的列表对象。</figcaption>
  <img src=Redis设计与实现.assets/image-20210929111815730.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-5 使用双端链表实现的列表对象。</figcaption>
</figure>


### 1.2.3 哈希对象

当敲入 `HMSET profile name "Tom" age 25 career programmer`:

<figure>
  <img src=Redis设计与实现.assets/image-20210929114152371.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-6-1 使用压缩列表实现的哈希对象。</figcaption>
  <img src=Redis设计与实现.assets/image-20210929114203569.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-6-2 压缩列表具体内容。先键后值顺序储存。</figcaption>
  <img src=Redis设计与实现.assets/image-20210929114316726.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-7 使用 hashtable 实现的哈希对象。</figcaption>
</figure>


### 1.2.4 集合对象

<figure>
  <img src=Redis设计与实现.assets/image-20210929115629153.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-8 使用整数集合实现的集合对象。</figcaption>
  <img src=Redis设计与实现.assets/image-20210929115642667.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-9 使用 hashtable 实现的集合对象。</figcaption>
</figure>


### 1.2.5 有序集合对象

* 为什么有序集合实现需要同时用到跳跃表和字典？

  回答之前，不如先看下如果单独使用跳跃表或者字典是否满足需求。

  单独使用字典，操作复杂度：

  * 查找成员分值O(1)；
  * 范围性操作O(nLOGn)；字典无序，面对 *ZRANGE、ZRANK* 等范围查找命令，只能遍历且排序。

  单独使用调表：

  * 查找成员分值O(LOGn);
  * 范围查找O(LOGn)；

  所以，Redis 用跳跃表组织节点，方便范围查找；而用字典记录每个节点分值。

<figure>
  <img src=Redis设计与实现.assets/image-20210929132458900.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-10-1 ziplist  编码的有序集合对象。</figcaption>
  <img src=Redis设计与实现.assets/image-20210929132510591.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-10-2 压缩列表的填充元素和对应分值，按照分值排序。</figcaption>
  <img src=Redis设计与实现.assets/image-20210929132535819.png alt="img" style="zoom:50%;">
  <figcaption>Fig. 1-11 使用 字典和调表实现的有序集合对象。</figcaption>
</figure>


## 1.3 编码方式

### 1.3.1 简单动态字符串 SDS（simple dynamic string）

#### 结构定义

```c
struct sdshsr {
  int len;  // 记录已经使用的字节的数目，其值等于 SDS 所保存字符串的长度。
  int free;  // buf 数组剩余可用空间
  char buf[];  // 字节数目，用于保存字符串
};
```
<figure>
  <img src=Redis设计与实现.assets/image-20210929142926021.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-12 SDS 示例。注意 len 是字符串长度（'\0'不算在内），free 是字节数。</figcaption>
</figure>
#### **SDS 的优越性**

与 C 字符串相比，SDS

* 常数复杂度获取字符串长度；

* **二进制安全**；

  C 字符串只能保存文本数据，SDS 能存储二进制数据。

* 杜绝缓冲区溢出；

  写入时会先判断 SDS 对象的大小，根据需要扩展。

* 减少修改字符串时带来的内存重分配次数

  拼接或者缩短字符串涉及到空间的分配与释放，Redis 尽量避免系统调用次数：

  * 优化 SDS 字符串增长操作 - 空间预分配

    如果 buf 空间不足，Redis 将分配额外的空间，为未来插入做预备。额外分配的空间大小取决于 SDS 的 len 字段。

  * 优化字符串缩短操作 - 惰性空间释放

    字符串被缩短时，程序并不立即回收内存，而是暂存在 free，预备后续使用。

* 兼容部分 C 字符串函数。

  毕竟 SDS 中的 buf 数组会自动在最后加上'\0'。

## 1.3.2 双端链表 linked list

#### 结构定义 - 链表节点与双端链表

```c
struct listNode {
  listNOde* prev;  // 前置节点
  listNode* next;  // 后置节点
  void* value;  // 节点数据
};
struct list {
  listNode* head; // 表头节点
  listNode* tail;  // 表尾节点
  unsigned long len;  // 存储节点数目
  void *(*dup) (void* ptr);    // 节点复制函数
  void *(*free) (void* ptr);   // 节点释放函数
  void *(*match) (void* ptr);  // 节点对比函数
};
```

<figure>
  <img src=Redis设计与实现.assets/image-20210929145341986.png alt="img" style="zoom:67%;">
  <figcaption>Fig. 1-13 双端链表。</figcaption>
</figure>

#### 特点

双端 + 无环 + 首尾指针 + 带长度 + 多态（节点数据类型不需要一致）

## 1.3.3 字典 dict

#### 结构定义 - 哈希表节点、哈希表与字典

```c
// 哈希表
struct dictht {
  dictEntry **table;  // 哈希表数组
  unsigned long size;  // 哈希表大小
  unsigned long sizemask;  // 哈希表大小掩码，用于计算索引值，总是等于 size - 1
  unsigned long used;  // 该哈希表已有节点的数目
}

// 哈希表节点
struct dictEntry {
  void *key;  // 键
  union {
    void* val;
    uint64_t u64;
    int64_t s64;
  } v;
  dictEntry *next;  // 指向下个哈希表节点，形成链表。主要是为了解决哈希冲突。
}

// 字典
struct  dict {
  dictType *type;  // 类型特定函数，比如可以传入计算哈希值的函数，murmurhash2 算法
  void *private;  // 私有数据，type 函数的可选参数
  dictht ht[2];  // 哈希表
  int rehashidx;  // rehash 索引，rehash 进行中则值为 -1；
}
```

<figure>
  <img src=Redis设计与实现.assets/image-20210929152651666.png alt="img" style="zoom:50%;">
  <figcaption>Fig. 1-14 一个键冲突了的字典对象。</figcaption>
  <img src=Redis设计与实现.assets/image-20210929153436900.png alt="img" style="zoom:50%;">
  <figcaption>Fig. 1-14 一个键冲突了的字典对象。</figcaption>
</figure>
#### 关键特性

* 解决键冲突（collision）

  Redis 使用链地址法（separate chaining）解决哈希冲突问题。

  哈希表节点的 next 指针，用于构成一个链表，发生冲突时挂在链表上即可。

* rehash

  随着操作的进行，哈希表上的节点可能过多或者过少，为了让哈希表的负载因子在一个合理的范围之内，程序需要对哈希表进行相应的扩张或者收缩。
  
  rehash 首先为 ht[1] 分配表空间（大小取决于是想扩张或者收缩），然后对于 ht[0] 上的每个节点进行 rehash。当所有节点搬迁完毕之后，交换 ht[0], ht[1]，释放 ht[0]。
  
* 渐进式 rehash
	
  实际上 rehash 并不是一次性完成的，如果表过大，rehash 庞大的计算量可能导致服务不可用。
  
  渐进哈希同时维持两个 ht[0] 和 ht[1]。

## 1.3.4 跳跃表 SkipList

跳跃表是一种有序的数据结构，通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。

详细的解读可以参考这篇[文章](https://www.jianshu.com/p/09c3b0835ba6)，写的非常漂亮。

### 结构定义 - zskiplistnode 与 zskiplist

zskiplistnode 提供了节点信息，比如表头、表尾、最高层和节点数目。
<figure>
  <img src=Redis设计与实现.assets/image-20210929163047300.png alt="img" style="zoom:50%;">
  <figcaption>Fig. 1-15 zskiplist 与其上的 zskiplistnode 节点，注意到表头是一个虚拟节点。</figcaption>
</figure>

整体结构非常巧妙，

* 每个 Level 是一个有序链表，除了 Level 0上放的是真实的数据以外，其他节点存储的节点指针。

* 一个节点会出现在多个 Level 上，并且这些Level 总是连续的，从零开始的。

  任何节点必定出现在 Level 0 上，这是为了保证必定有一层能遍历到所有节点。

  一个节点的层高 L 是随机决定的，L 越大越难出现。这是为了保证越上层的 Level 的链表上的节点越小。


  ```C++
  // 每个节点是一个在 1 - 32 之间的随机的一个整数。
  int randomizeLevel(double p, int lmax) {
      int level = 1;  // 保证至少有一层
      Random random = new Random();
      while (random.nextDouble() < p && level < lmax) {  // p 决定层高，p 越大，可能的层数就越高。
          level++;
      }
      return level;
  }
  ```

* 查找流程

  查找的复杂度为 log(N)，最差情况下是 N。

  1）总是从最上层开始查找；（最上层入口存放在 ziplist）；

  2）如果当前层的当前节点的下一个节点比查找的值大，则下移到当前节点的下一层；

  3）否则继续向右边查找。

<figure>
  <img src=Redis设计与实现.assets/image-20210929170744452.png alt="img" style="zoom:50%;">
  <figcaption>Fig. 1-16 查找的值为25的查找流程。</figcaption>
</figure>
### 复杂度分析

假设 p 决定某个节点是否增加一层（就是上面`randomizeLevel` 函数），下面进行简单的[逻辑推演](https://15721.courses.cs.cmu.edu/spring2018/papers/08-oltpindexes1/pugh-skiplists-cacm1990.pdf)：

* 空间复杂度

  假设抽样的概率是 p，如果 level 0 有 N 个元素，则 level k 层节点个数是 N * p^k，

  因此空间复杂度是：`N * (p + p^2 + p^3 + p^4 + ...) = N * p / (1 - p)`，也就是 **O(N) 的空间复杂度**。

  > 注意，这里其实是 N 个指针（而非真正数据）的内存消耗。

* 单个节点期望层高

  对单个节点而言，层高为 1 的概率为 (1-p)，层高为 2 的概率为 (1-p)\*p，层高为 k 的概率为 (1-p)\*p^(k-1) ..

  因此单节点层高期望为 $\sum_{k=1}^{\infty}{k * (1-p)*p^{k-1}} = 1/(1-p)$;

* 查找复杂度

  最高层只有一个点，那么就有 `N*p^(K-1) = 1` => $ K = 1 + \log_{1/p}N$

  平均查找长度就是查找的时间复杂度。

  为了便于分析，我们将查找过程逆过来， 向上和向左回到查找的节点。

  查找层 L(n) = $\log_{1/p}{N} = \log_{p}{1/N}$，有 $N * p^{L(n)} = 1$ 个节点。

  假设 C[k] 为向上走 k 层的路径期望值，则有 C[0] = 0, C[k] = (1-p) * (1 + C[k]) + p * (1+C[k-1])；

  则可以推出 C[k] = k / p；

  因此查找复杂度为：向上走的路径 + 向左走的路径 <=$K / p = 1/p + log_{1/p}(N)$，也就是 O(logN) 复杂度。

  插入和删除同理。

### 为什么要使用跳跃表（而不是树）实现有序集合 

Redis 作者 Salvatore Sanfilippo 的[原话](https://news.ycombinator.com/item?id=1171423)：

1. 跳跃表并没有很消耗内存（"not memory intensive"）。调整节点层数概率 p 能使得跳跃表比 BST 占用更少内存。
2. 利用局部缓存（cache locality）。对于遍历操作（ZRANGE，ZREVRANGE）调表的局部缓存至少不会比树差；
3. 实现更加简单，更容易 debug。

## 1.3.5 整数集合

### 结构定义

整数集合实际上是一个从小到大的有序数组，并且不包含任何重复项。

```c
struct intset {
  uint32_t encoding;  // 编码方式
  uint32_t length;  // 集合包含的元素数目
  int8_t contents[];  // 保存元素的数组
};
```

### 整数集合的升级

整数集合支持三种类型：int16_t、int32_t 和 int64_t 的整数值。

如果插入的类型比之前存在的类型高，那么需要**升级**：申请新的空间，转换类型。

注意一旦升级之后，就不能降级了。

升级有两个好处：

* 提升灵活性

  三种不同类型可以放在一个数组内；

* 节约内存

  必要时才升级，节省空间。

## 1.3.6 压缩列表

压缩列表是 Redis  为了节约内存而开发出来的，是由一系列特殊编码的**连续内存块**组成的数据结构。

一个压缩列表可以包含任意多个节点，每个节点可以保存一个字节数组或者整数值。

每个压缩节点由三部分构成：

* previous_entry_length 当前节点的前一节点的长度，可以为 1 个或者 5 个字节；
* encoding 当前节点存储的数据类型：整数或者字节数组；
* content 真正的数据；

### 连锁更新

插入了一个很大的节点，原先节点的 previous_entry_length 被迫从 1 字节升级到 5 字节，导致整个压缩列表都需要更新。

导致插入的复杂度由 O(N) 变为 O(N^2)。
