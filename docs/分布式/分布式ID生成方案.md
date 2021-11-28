# 分布式唯一ID
---

```md
author: WuDG
datetime： 23:40:40 星期二 2021年11月26日
position： 杭州
```

> 全局唯一、高性能、好接入、递增

## 1. 雪花算法
```properties
64 = 1（正数 0） + 41（毫秒时间戳） + 10（机器ID） + 12（序列号）
使用时长：(2^41 - 1)/(1000*60*60*24*365) = 69年
节点数：2^10（区分步长）
序列号：2^12-1 = 4095 （同一毫秒生成ID数量）
```

## 2. UUID
```properties
String uuid = UUID.randomUUID().toString().replaceAll("-","");
简单、无序、无具体业务含义、过长
```

## 3. 基于数据库自增
```properties
基于数据库的 auto_increment 自增ID
需要一个单独的 MySQL 实例用来生成ID
访问量激增时 MySQL 本身就是系统瓶颈，不支持
```

## 4. 基于数据库集群
```properties
在上面单个数据库基础上
解决数据库单点压力
通过初始值和步长生成唯一ID
扩容困难
```

## 5. 基于数据库的号段模式
```properties
主流之一
从数据库批量获取自增ID
CREATE TABLE id_generator (
  id int(10) NOT NULL,
  max_id bigint(20) NOT NULL COMMENT '当前最大id',
  step int(20) NOT NULL COMMENT '号段的布长',
  biz_type    int(20) NOT NULL COMMENT '业务类型',
  version int(20) NOT NULL COMMENT '版本号',
  PRIMARY KEY (`id`)
) 
```

## 6. 基于redis模式
```properties
利用 redis 的 incr 命令实现ID的原子性自增
考虑持久化问题
RDB会定时打一个快照进行持久化，假如连续自增但redis没及时持久化，而这会Redis挂掉了，重启Redis后会出现ID重复的情况。

AOF会对每条写命令进行持久化，即使Redis挂掉了也不会出现ID重复的情况，但由于incr命令的特殊性，会导致Redis重启恢复的数据时间过长。
```

## 7. 百度（uid-generator）
```properties
基于雪花算法
支持自定义时间戳、工作机器ID和序列号等位数
配合数据库使用，新增一个 WORKER_NODE 表，应用启动时向表中插入一条记录，插入成功后返回自增ID就是机器的workId
64 = 1（正数 0） + 28（秒） + 22（机器ID） + 13（序列号）
```

## 8. 美团（Leaf）
```properties
同时支持号段模式和雪花算法模式
号段模式：
CREATE TABLE `leaf_alloc` (
  `biz_tag` varchar(128)  NOT NULL DEFAULT '' COMMENT '业务key',
  `max_id` bigint(20) NOT NULL DEFAULT '1' COMMENT '当前已经分配了的最大id',
  `step` int(11) NOT NULL COMMENT '初始步长，也是动态调整的最小步长',
  `description` varchar(256)  DEFAULT NULL COMMENT '业务key的描述',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '数据库维护的更新时间',
  PRIMARY KEY (`biz_tag`)
) ENGINE=InnoDB;
雪花算法模式：
依赖于 ZK，Lead 中国呢 workId 基于 zk 的顺序ID来生成
```

## 9. 滴滴（TinyId）
```properties
原理如 Leaf 的号段模式

```
