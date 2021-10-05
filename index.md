## haydengao的个人主页

> I seem to have been only like a boy playing on the sea-shore, and diverting myself in now and then finding a smoother pebble or a prettier shell than ordinary, whilst the great ocean of truth lay all undiscovered before me.  
>
> -- Louis Trenchard More, *Isaac Newton: A Biography* (New York: Scribner's, 1934), p. 664.

本人摘抄整理知识的地方，督促自己读书写字。

主要分为两类：

**[读书笔记](https://github.com/haydengaoCN/haydengaoCN.github.io/tree/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0) 书籍摘抄，主题学习**。

1. [Linux 高性能服务器编程](https://github.com/haydengaoCN/haydengaoCN.github.io/tree/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B) 这本书从 TCP/IP  协议族讲起，然后讲 socket 编程，最后给出服务器经典的框架模型。受益最深的是对三个 I/O 复用函数的讲解，帮我解了多年来的困惑。

2. [MySQL_InnoDB存储引擎](https://github.com/haydengaoCN/haydengaoCN.github.io/tree/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/MySQL_InnoDB%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E) 摘抄自被无数人奉为神书的「MySQL技术内幕-InnoDB存储引擎」。虽然之前零散的看过讲解 MySQL 原理的文章，阅读这本书时还是觉得困难重重，难以理解。本书假定读者已经有了一定的基础知识，章节之间并没有明显的界限，算是知识的杂糅综合 （阅读时的错乱和困惑让我想起了大学时学习线性代数的日子）。推荐先快速阅读一遍，对各个知识块有个大概印象，第二遍再细读。

3. [Redis 设计与实现](https://github.com/haydengaoCN/haydengaoCN.github.io/tree/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0) 作者以 Redis 3.0 版本为基础，介绍 Redis 内部构造以及运作机制，全书共分为四大部分：

   [Part 1 数据结构与对象](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/part1-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E5%AF%B9%E8%B1%A1.md) 介绍了 Redis 的五大对象（字符串、列表、哈希、集合和有序集合对象）以及底层的数据结构实现（SDS、Linked List、Dict、Skip List、Int Set 和 ZipList）。

   <u>Part 2 单机数据库的实现</u> 

   ​	[数据库](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/9-%E6%95%B0%E6%8D%AE%E5%BA%93.md) Redis 数据库存放于内存之中，以字典的方式组织索引。介绍了过期键的删除策略：定时删除、惰性删除和定期删除。

   ​	[Redis 持久化](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/Redis%E6%8C%81%E4%B9%85%E5%8C%96_chap10-11.md) Redis 提供了快照保存方式 RDB 和命令记录方式 AOF 两种持久化手段。

   ​	[Redis 客户端和服务端](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/Redis%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%B8%8E%E6%9C%8D%E5%8A%A1%E7%AB%AF_chap12-13.md) 一条命令从客户端发送到服务端的流程。

   ​	[事件](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/12-%E4%BA%8B%E4%BB%B6.md) Redis 服务端是由事件驱动的服务，事件分为文件事件和时间事件；文件事件是对 socket 状态的抽象，时间事件指的是需要	定期执行的操作（实际上只有一个 `serverCron` 函数)。本章更为重要的是介绍了 Redis 的网络模型：单线程的 Reactor 架构。主  	线程既负责利用 I/O 多路复用函数处理网络通信，又负责调用事件处理器处理命令请求。

   [Part 3 多机数据库的实现](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/part3-%E5%A4%9A%E6%9C%BA%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E5%AE%9E%E7%8E%B0.md) 单个服务器毕竟内存和 QPS 都受限，为了提升性能和可用性，可以采用 Redis 多机数据库。

   * 复制 Replication：高可用 Redis 的基础。哨兵和集群都是在此基础上实现的高可用。实现数据多机备份以及读操作的负载均衡。
   * 哨兵 Sentinel：在复制的基础上，实现了自动化的故障转移。
   * 集群 Cluster：读写操作负载均衡，存储能力不再受限于单机。是较为完善的高可用方案。
   
   [Part 4 Redis 独立功能](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/note-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/part4-Redis%E7%8B%AC%E7%AB%8B%E5%8A%9F%E8%83%BD.md) 介绍了 Redis 的独立功能：发布和订阅、事务、LUA 脚本、排序、二进制位数组、慢查询日志和监视器功能。



**[浅斟低唱](https://github.com/haydengaoCN/haydengaoCN.github.io/tree/main/sign-%E6%B5%85%E6%96%9F%E4%BD%8E%E5%94%B1) 离散知识点，记录工作中用到的组件背后原理**。

1. [关系型数据库与非关系型数据库](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/sign-%E6%B5%85%E6%96%9F%E4%BD%8E%E5%94%B1/1-%E5%85%B3%E7%B3%BB%E5%9E%8B%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%8E%E9%9D%9E%E5%85%B3%E7%B3%BB%E5%9E%8B%E6%95%B0%E6%8D%AE%E5%BA%93.md) 二者的简单比较。
2. [RocksDB 磐石方且厚](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/sign-%E6%B5%85%E6%96%9F%E4%BD%8E%E5%94%B1/2-RocksDB%E4%BD%8E%E5%90%9F.md) 关键字：LSM，SST，WAL。
3. [Redis 初探](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/sign-%E6%B5%85%E6%96%9F%E4%BD%8E%E5%94%B1/3-Redis%E5%88%9D%E6%8E%A2.md) 
4. [ElasticSearch 简述](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/sign-%E6%B5%85%E6%96%9F%E4%BD%8E%E5%94%B1/4-ElasticSearch%E7%AE%80%E8%BF%B0.md) 关键字：全文检索，ES 整体结构，写入和搜索过程。
5. [RPC 只缘身在此山中](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/sign-%E6%B5%85%E6%96%9F%E4%BD%8E%E5%94%B1/5-RPC%E5%8F%AA%E7%BC%98%E8%BA%AB%E5%9C%A8%E6%AD%A4%E5%B1%B1%E4%B8%AD.md) 关键字：RPC，服务发现，序列化与反序列化。
6. [Pulsar 湖光秋月两相和](https://github.com/haydengaoCN/haydengaoCN.github.io/blob/main/sign-%E6%B5%85%E6%96%9F%E4%BD%8E%E5%94%B1/6-Pulsar%E6%B9%96%E5%85%89%E7%A7%8B%E6%9C%88%E4%B8%A4%E7%9B%B8%E5%92%8C.md) 关键字：消息中间件，计算和存储分离，分片存储。

-----------



牛顿爵士在自传中谦虚地把自己比喻成一个在知识的汪洋大海边玩耍的小男孩，不时为捡到一颗光滑的鹅卵石或一块漂亮的贝壳而欣喜若狂。

牛顿爵士烦恼的是真理之海只被揭露了一小部分，我的烦恼在于如何安置这不时捡到的鹅卵石和贝壳。如果一个零散的知识点就是一颗光滑的鹅卵石，只有一颗的话你自然可以随意将其揣在兜里；但是随着鹅卵石不断增多，口袋就装不下了，促使你考虑如何分类和存储。也就是说，当面对着奔涌的、不断接触到的信息，如何高效的理解和归纳？

窃以为弯腰拾贝壳之前，更重要的是先建立自身知识宫殿的骨架。学习一个新的事物时，最好先搭建一个知识的框架，然后再丰富其骨肉。换句话说，学习应该是一个自顶向下的过程，先学习高层的抽象，然后再补充细节，理解抽象语句下的真实内涵。

我理解自顶向下的学习过程至少可以避免走向两个弯路：

一是过早深入，囿于细节。比如在学习 InNoDB 的重做日志 RedoLog，核心是要明白事务提交之前会先将本次操作刷新到位于磁盘的 RedoLog， 以保证数据的持久性 （**D**urable）。至于 RedoLog 文件格式、Group  Commit 以及和 binlog 交互等问题，或许可以暂时先放着。

二是不成条理，缺乏理解。在比较 MySQL 和 NoSQL 时，常有人说前者是支持事务的，后者不支持。而事实上数据库是否支持事务取决于存储层的存储引擎，比如一个表使用了 InNoDB，那么这个表就支持事务操作；而同一个数据库中另一个表使用了 MyISAM ，那么这个表就不支持事务。如果对数据库的基本抽象（server 层 + 插件式的存储引擎）都没有，那么就无法鉴别这类似是而非的论断，更谈不上有自己的理解。

那么如何建立对陌生事物的高度抽象呐？无它，书籍是人类进步的阶梯。

书籍是成体系的知识的代表，它的目录就是高度抽象的结果。作者将主要知识点分类和归纳，十分慷慨的将所涉内容的高层抽象呈现给读者。建立知识框架后，再去阅读书本具体内容，便不会觉得突兀和凌乱。

写在这里，也是给自己的一个提醒，多看书，少刷微博。
