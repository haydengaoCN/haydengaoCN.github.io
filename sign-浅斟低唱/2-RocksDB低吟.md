> 最近的项目中使用到了RocksDB，这里介绍下其整体架构和核心组件。

# 0-RocksDB 简介

RocksDB 是一款基于 [LevelDB](https://github.com/google/leveldb) 开发的支持任意字节的 K-V 对的 LSM-Tree 存储引擎。

其同时支持 point lookups 和范围查找并且支持 ACID。

RocksDB 能够运行在 SSD、硬盘、RAMfs 和远程存储等多个环境和各种压缩算法。

# 1-核心组件

## 1.1 整体架构

RocksDB由三个基础的组件构成：MemTable、SST （string sorted table）和 WAL（Write Ahead Log）。

写入时RocksDB会先写入位于内存中的MemTable（而非直接写入到磁盘中）和顺序写入磁盘文件 logfile。

当 MemTable 被写满时，RocksDB 才会将内存中的内容写入到 SST 文件中，SST 文件中有多层并且其内容是按照Key的某种顺序存放的，除了L0之外，L1-Ln 的 SST 文件包含的 key 的范围都不会互相覆盖。

当数据被写入到SST文件后就可以删掉对应的 logfile。

多层 Level 的 SST 文件就构成了一棵  LSM 树。

LSM 树特点：

1）除了 Level 0 以外，Level 的数据按照 KEY 有序排列，并且被划分为多个 SST 文件；

2）每个 Level 中的 SST 文件数目不同，下层与上层文件数目之比称之为扇出；

3）上层文件在适当的时机会合并到下层文件，这个过程称之为 Compaction；

3）一个 KEY 可能出现在多个 LEVEL 中，但是上层的版本总是高于下层的版本（就是上层数据更加新）；

<figure>
  <img src="2-RocksDB低吟.assets/119747261-310fb300-be47-11eb-92c3-c11719fa8a0c.png" alt="img" style="zoom: 25%;">
  <figcaption>Fig.1-1 RocksDB 整体架构。写入时先存在内存中，然后再写入磁盘；每个 Column Family 各自有 MemTable 和 SST 文件，如虚线框所示。</figcaption>
</figure>



## 1.2 MemTable

当更改数据时，数据暂时存储在内存中的一块被称之为 MemTable 的数据结构中。MemTable 被写满时，MemTable 会被标记为 immutable (被称之为 **Immatable MemTable**）并且被一块新的 MemTable 代替。当积攒了一定数目的 MemTable 时，background 线程就会将数据刷新到磁盘，MemTable 就可以被清除。

可见 MemTable 存放了 RocksDB 最新的数据，读操作总是会先查询 MemTable 然后再查询 SST 文件。

MemTable 中的数据是有序的，默认的实现方式是调表 Skiplist，此外还支持HashSkipList，HashLinkList 或者 Vector。

> HashSkipList：根据 key 的前缀将数据放到哈希桶中，桶中的数据还是根据 Skiplist 组织。
>
> 查找时先根据 KEY 前缀找到桶，然后在调表上查询。适合范围查找（有固定前缀）。
> HashLinkList：和HashSkipList类似，但是桶中的数据用链表组织。适合范围查找并且数量较少的数据。

[MemTable刷新的时机以及实现方式的比较](https://github.com/facebook/rocksdb/wiki/MemTable)

## 1.3 WAL（Write Ahead Log）

前面提到过，RocksDB每次的更新会被写入两个地方：1）内存中的MEMTable 和 2）位于磁盘的 write ahead log 。

WAL 记录了每次更新并且及时刷新到磁盘，如此保证了当进程宕机的时候能够从 WAL 恢复数据。

**WAL 的生命周期**：

WAL 被创建于 1）一个新 DB 打开的时候；2）某个 Column Family 被刷新到磁盘上。用户主动将 Column Family 中的数据刷新到 SST 文件时，新的 WAL 就会被创建，后续的更新会被写到新的 WAL。 当所有的数据被刷新到 SST 时，旧的 WAL 就可以被删除或者归档（archive 是为了不同实例时间的 replication）。

**WAL 如何判断 Column Familys 的数据已经完全刷新到磁盘**？

比较 Column Familys  和 WAL 的 LSN （Largest Sequence Number），如果前者大于后者就说明 WAL 数据已经被刷新到磁盘了。

一个有趣的问题是既然都要写磁盘，**为什么不直接把真正的数据刷新到磁盘**？

这并不是南辕北辙。写磁盘上的 WAL 可以顺序写，但是写数据确是随机的。因为磁盘上的数据是按照 KEY 的一定规则顺序存储的，随机的数据更新带来随机的磁盘插入。WAL 只是记录更新的日志，只要继续插入在前方即可。

## 1.4 SST （Sorted String Table）

### **SST 组织方式**

SST 的组织架构就是 LSM Tree，一切为了更好的查找性能。

每个 Level 包含多个文件，每个文件包含一定范围的 KEY` [FileMetaData.smallest, FileMetaData.largest]`;

Level 0 中的 SST 文件 RANGE KEY 是可以重复的，SST 文件按照 Flush 的时间进行排序。查找时必须 Level 0 中的所有文件。

Level x ( x >= 1) 中的 SST 文件保证 RANGE  KEY  不重叠，并且按照 RANGE KEY 排序。因此查找时可以用二分查找。

一次查询可能需要查找多个Level，RocksDB 采用了 [fractional cascading](https://en.wikipedia.org/wiki/Fractional_cascading) 加快查找速度。

```tex
                                         file 1                                          file 2
                                      +----------+                                    +----------+
level 1:                              | 100, 200 |                                    | 300, 400 |
                                      +----------+                                    +----------+
           file 1     file 2      file 3      file 4       file 5       file 6       file 7       file 8
         +--------+ +--------+ +---------+ +----------+ +----------+ +----------+ +----------+ +----------+
level 2: | 40, 50 | | 60, 70 | | 95, 110 | | 150, 160 | | 210, 230 | | 290, 300 | | 310, 320 | | 410, 450 |
         +--------+ +--------+ +---------+ +----------+ +----------+ +----------+ +----------+ +----------+
```



### **SST 具体格式**

* data block

  存储键值对数据，所有的数据按照 KEY 有序排列，然后被划分到一系列 blocks 中。

* meta block

  meta block 紧接着 data  block，用于数据管理。

  **filter block** 使用布隆过滤器快速确定 KEY 是否存在。

  **index block** 存放着 data blocks 中 KEY 的范围，二分查找确定 query key 的位置。

  **metaindex block** 是一个 map，meta 名字映射到具体的 meta block。

```tex
<beginning_of_file>
[data block 1]
[data block 2]
...
[data block N]
[meta block 1: filter block]                  (see section: "filter" Meta Block)
[meta block 2: index block]
[meta block 3: compression dictionary block]  (see section: "compression dictionary" Meta Block)
[meta block 4: range deletion block]          (see section: "range deletion" Meta Block)
[meta block 5: stats block]                   (see section: "properties" Meta Block)
...
[meta block K: future extended block]  (we may add more meta blocks in the future)
[metaindex block]
[Footer]                               (fixed size; starts at file_size - sizeof(Footer))
<end_of_file>
```



## 1.5 Compaction

compaction 算法刻画了 LSM Tree 的形状。

合并 Level n 就是将 Level (n-1) 的数据合并到 Level n，Level n 的数据需要重写（读写放大问题）。

归并的策略有：

1）**Leveled Compaction**；

2）**Universal Compaction**；主要是针对 L0，考虑的是一次性合并多少个 SST 文件到下层。

3）**IFO Compaction**；在这种模式下，所有的文件都在 L0，当文件总数达到阈值时，就删掉最老的文件。

# 2-读写过程

## 读

1）在 MemTable 中查找，无法命中转到下一步；

2）在 immutable_memtable 中查找，查找不中转到下一步；

3）在第0层 SSTable 中查找，无法命中转到下一流程； 

> 对于L0 的文件，RocksDB 采用遍历的方法查找，所以为了查找效率 RocksDB 会控制 L0 的文件个数。每个 Memt able 跟 SST 都会有相应的 Bloom Filter 来加快判断 Key 是否可能在其中，当判断 Key 可能在其中时，就会在 Memtable 或者 SST 中进行查找。

4）在剩余SSTable中查找。对于 L1 层以及 L1 层以上层级的文件，每个 SSTable 没有交叠，可以使用二分查找快速找到 key 所在的 Level 以及 SSTfile。

## 写

循环检查 DB 状态;

1）如果当前 memtable 的 size 未达到阈值 write_buffer_size(默认4MB)，则允许写入; 如果 memtable 的 size 已经达到阈值，但 immutable memtable 仍然存在，则等待 compaction 将其 dump 完成；

2）memtable 已经写满，并且 immutable memtable 不存在，则将当前 memetable 置成 immutable memtable，产生新的 memtable 和 log file，主动触发 compaction，允许该次写。

3）数据先写入 WAL，成功后转到下一步；

4）数据写入 memetable；

# 3-存在的问题

- 读写放大严重；
- 应对突发流量的时候削峰能力不足；
- 压缩率有限；
- 索引效率较低；

# 参考文献

1. [RocksDB Wiki](https://github.com/facebook/rocksdb/wiki)
2. [以RocksDB为代表的LSM tree 的学习与总结](https://lxkaka.wang/rocksdb-lsm/#lsm-tree-%E7%AE%80%E4%BB%8B)；
3. [图解NoSQL的江湖称霸之路](https://database.51cto.com/art/202108/679289.htm#topx);

