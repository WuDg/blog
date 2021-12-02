# redis

---

```md
author: WuDG
datetime： 10:4040 星期二 2021年8月3日
position： 杭州
```

## 一、redis 概述 & redis 特点

> Redis（Remote Dictionary Server）远程数据服务，是一种支持 key-value 等多种数据结构的存储系统。可用于缓存，事件发布或订阅，高速队列等场景。支持网络，提供 string，hash，list，set，zset 等数据结构，基于内存，可持久化

* 丰富的数据类型：适用场景多
* 内存存储：快
* 持久化：数据安全

## 二、数据结构

> redis 是 key-value 数据库，key 只能是 string，value 可以是 string、hash、list、set、 sorted set

* string：最大存储 512m（set key_name value），信息缓存、计数器、分布式锁
* hash：适合存储对象，key-value 集合(hset key_name field value)，购物车（用户id为key，商品id为field，商品数量为value）
* list：字符串列表，有序可重复（lpush key_name value1.. value10），是一个分页做定时排行榜
* set：字符串无序不可重复集合（sadd key_name value...），收藏夹
* sorted set：有序集合（zadd key_name score value），实时排行榜

## 三、redis 持久化策略

> 将内存中数据存储到磁盘中

* RDB：快照存储
* AOF：记录写操作，追加到日志文件中，服务器重启时重新执行这些命令
* 不使用持久化
* RDB&AOF：同时开启两种持久化，redis重启时优先载入 AOP 文件来恢复原始数据，AOF中数据比RDB更加完整

## 四、redis 线程模型

redis 内部使用文件处理器，单线程架构。采用 IO 多路复用机制同时监听多个 socket，根据 socket 上的事件选择对应的事件处理器进行处理
单线程高效率：

* 纯内存操作
* 核心是基于非阻塞的 IO 多路复用机制
* 单线程避免了多线程上下文切换

## 五、常见问题

* 雪崩：缓存中大批量 key 过期，请求直接落到数据库。使 key 过期时间均匀，加互斥锁，用不国旗
* 穿透：恶意请求系统中不存在的数据。使用布隆过滤器（爬虫系统url去重、垃圾邮件过滤、黑名单），返回空对象
* 预热：系统上线后将缓存数据直接加载到缓存系统，避免用户请求直接落到数据库再写到缓存。工程启动时进行加载缓存动作，设置一个定时任务脚本，先保证热点数据提前加载到缓存
* 击穿：某热点 key 失效瞬间，请求直接落到数据库。增加回血缓存互斥锁，热点 key 永不过期
* 降级：指缓存失效或者服务器挂掉，不直接访问数据库，直接返回默认值或者访问服务的内存数据，避免数据库遭受巨大压力

## 六、内存淘汰机制

> redis 缓存不足时，通过淘汰旧数据来处理新加入数据的策略

* noeviction：默认策略，对于写请求直接返回错误
* allkeys-lru：所有 key 最近最少使用
* volatile-rlu：设置了过期时间的 key 最近最少使用
* allkeys-random：所有 key 中随机淘汰
* volatile-random：从设置了过期时间的 key 中随机淘汰
* volatile-ttl：在设计了过期时间的 key 中根据过期时间淘汰最早过期的 key
* allkeys-lfu：所有的 key 中，最少使用频率，4.0开始支持
* volatile-lfu：设置了过期时间的 key 中，最少使用频率，4.0开始支持

## 七、redis事务机制

> MULTI 开启，操作命令加入队列，EXEC 命令开始顺序执行。如果执行报错，不会回滚

* WATCH：为 redis 提供 check-and-set（CAS）行为。被 WATCH 的 key 如果被改动，则 EXEC 时会返回 nil-reply
* MULTI：开启一个事务
* UNWATCH：取消 WATCH 命令所有的 key
* DISCARD：放弃事务，事务列表被清空，并退出事务状态
* EXEC：负责执行事务中所有命令

## 八、redis 主从复制

> 指将一台 redis 服务器数据复制到其他 redis 服务器。前者为主节点（master），后者为从节点（slave）。数据的复制是单向的，只能由主到从

主从复制作用：

* 数据荣誉：热数据备份
* 故障恢复：主节点故障，可以由从节点提供服务
* 负载均衡：读写分离，主节点负责写数据，从节点负责读数据
* 高可用：是哨兵和集群的基础

实现原理：

> 三个阶段：连接建立阶段、数据同步阶段、命令传播阶段

## 九、哨兵（Sentinel）原理

> 是 redis 高可用的实现方案，管理多个 redis 实例，实现对 redis 的监控、通知、自动故障转移

问题引入：redis 主从负责模式下，一旦主节点故障无法提供服务，需要手动将从节点晋升为主节点，还需要通知客户端更新主节点地址，这个故障处理方案是不可取的

## 十、 memcache & redis 区别

* memcache 纯内存，数据量不能超过内存，redis 支持持久化
* memcache 只支持 string，redis 支持多种

## 十一、客户端

* Jedis
* Redisson
* Lettuce