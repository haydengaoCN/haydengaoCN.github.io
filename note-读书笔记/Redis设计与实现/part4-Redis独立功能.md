# 1 - 发布和订阅

Redis 发布和订阅相关的命令：*PUBLISH*、*SUBSCRIBE*、*PSUBSCRIBE*、*UNSCRIBE*、*PUNSCRIBE* 等命令组成。

`p` 开头的命令指的是 `pattern`，也就是模式匹配。

```bash
SUBSCRIBE "news.it"  # 客户端订阅了该频道
PUBLISH "news.it" "hello"  # 另一个客户端发送，那么所有订阅该频道的客户端都能接收到该消息
```
<figure>
  <img src=Redis设计与实现.assets/image-20211004103927988.png alt="img" style="zoom:80%;">
  <figcaption>Fig. 1.1 通过订阅匹配，确定能接收到消息的客户端。</figcaption>
</figure>



## 相关数据结构

1）频道订阅 *SUBSCFRIBE* 与 退订*UNSUBCRIBE*

`redisServer` 的成员变量 `pubshb_channels` 是一个字典，维持了 channel 频道到 client(s) 客户端的映射关系。

* *SUBSCFRIBE* 就是将 channnel 和 client 的信息添加到字典中；
* *UNSUBSCFRIBE* 会移除相关 channel  和 client。

2）模式订阅 *SUBSCFRIBE* 与 退订*UNSUBCRIBE*

`redisServer` 的成员变量 `pubshb_patterns` 是一个链表，记录 channel 和 clients 的匹配关系。

这里的 channel 以通配符的形式表示，不好直接映射。

3）发布 *PUBLISH*

* 访问 `pubshb_channels` 向所有订阅该频道的客户端发送消息；
* 访问 *pubshb_patterns* 向所有匹配频道的订阅者发送消息；



#  2 - 事务

Redis 通过将一系列请求命令打包，然后一次性、顺序地执行；执行过程中，服务器不会去执行其他客户端的命令。

相关的命令有：*MULTI* 、*EXEC* 和 *WATCH* 。

大概的过程就是：*MULTI* 标志事务开始，然后命令入队，最后执行。

## 重点总结

事务提供了一种将多个命令打包，然后一次性、有序地执行的机制；

1）Redis 如何保证命令执行的顺序性？

​	多个命令会入队到一个事务队列中，按照先进先出的顺序执行；

​	这里的命令指的是在 *MULTI* 和 *EXEC* 之间客户端敲入的命令；

2）Redis 保证事务执行过程不会被打断

​	事务在执行过程中不会被中断，只有事务队列中所有命令执行完毕，事务才算执行结束；

3）Redis 如何保证事务的安全性？

​	可以通过 *WATCH* 命令保证事务安全性。

* *WATCH* 在 *EXEC* 命令执行前，可以监视任意数量的数据库键；
* 监视过程中，如果有任何针对被监视键的写操作，则将执行事务的客户端标记为 `REDIS_DIRTY_CAS`;
* *EXEC* 准备执行时，检查客户端的标记，如果为 `REDIS_DIRTY_CAS` ，则拒绝执行，事务执行失败。

Redis  事务的 ACID 特性。

1）Redis 总是具有 A、C 和 I 特性，即原子性、一致性和隔离性。

2）当服务器运行在 AOF 持久化模式下，且 appendfsync 选项的值为 always 的时候，事务也具有耐久性。

3）Redis 事务不支持回滚。



# 3 - LUA 脚本

* Redis提供了非常丰富的指令集，官网上提供了200多个命令。
* 但是某些特定领域，需要扩充若干指令原子性执行时，仅使用原生命令便无法完成。
* Redis 为这样的用户场景提供了 lua 脚本支持，用户可以向服务器发送 lua 脚本来执行自定义动作，获取脚本的响应数据。
* Redis 服务器会单线程原子性执行 lua 脚本，保证 lua 脚本在处理的过程中不会被任意其它请求打断。



# 4 - 排序

快排算法实现排序。



# 5 - 二进制位数组

Redis 支持 bit  级别的数据结构，并提供相关的操作命令。

Redis 使用 SDS 来保存数组。



# 6 - 慢查询日志

记录执行时间超过给定时长的命令请求。



# 7 - 监视器

通过执行 *monitor* 命令，客户端可以将自己变成服务器的监视器，实时接收并打印出服务器当前处理的命令请求的相关信息。
