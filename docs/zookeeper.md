# zookeeper

```md
author: WuDG
datetime： 11:0932 星期四 2021年8月5日
position： 杭州乐佳大厦/博彦
```
> 我曾七次鄙视自己的灵魂
第一次，当它本可进取时，却故作谦卑；
第二次，当它在空虚时，用爱欲来填充；
第三次，在困难和容易之间，它选择了容易；
第四次，它犯了错，却借由别人也会犯错来宽慰自己；
第五次，它自由软弱，却把它认为是生命的坚韧；
第六次，当它鄙夷一张丑恶的嘴脸时，却不知那正是自己面具中的一副；
第七次，它侧身于生活的污泥中，虽不甘心，却又畏首畏尾。 

## 一、zookeeper 入门

### 1.1 zookeeper 概述
> zk 是一个开源的分布式协调服务，将复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，提供给用户使用

zk 提供了高可用、高性能、稳定的分布式数据一致性解决方案。通常被用于实现数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、master 选举、分布式锁和分布式队列等功能

另外，zk 将数据保存在内存中，性能很nice

### 1.2 zk 特点
* 顺序一致性：客户端发起的事务，按照顺序被应用到 zk 中
* 原子性：事务请求，对集群中所有的及其应用情况是一样的，要么都成功，要么都失败
* 单一系统映像：zk 集群保证查询到的数据都是一致的
* 可靠性：一旦更改请求被应用，数据将持久化

### 1.3 应用场景
* 分布式锁：通过创建唯一节点获得分布式锁，当获得锁的一方执行完或者挂掉后就释放锁
* 命名服务：通过 zk 的顺序节点生成全局唯一 ID
* 数据发布/订阅：通过 Watch 机制，实现数据发布/订阅。数据被发布到 zk 的被监听的节点上，其他及其可以通过监听该节点来实现配置的动态更新

> 开源项目使用 zk：kafka、hbase、hadoop

## 二、zk 重点
> zk 主要是用来协调服务的，不是用来存储业务数据的，不能将大数据放在 zk 中


### 2.1 Data model（数据模型）
> 类 Linux 系统中文件系统，树形结构

### 2.2 znode（数据节点）
> zk 中最小数据单位

* 持久节点：持久化，直到被删除
* 临时节点：生命周期为 和客户端绑定（session），会话消失则节点被删除，临时节点只能做叶子节点，不能创建子节点
* 持久顺序节点：有序（如 /node1/app000000001）
* 临时顺序节点：有序

znode 数据结构
```json
{
    "stat" : "状态信息",
    "data" : "节点存放的内容"   
}
```

**stat 中数据信息**

cZxid:create ZXID，即该数据节点被创建时的事务 id
ctime:create time，即该节点的创建时间
mZxid:modified ZXID，即该节点最终一次更新时的事务 id
mtime:modified time，即该节点最后一次的更新时间
pZxid:该节点的子节点列表最后一次修改时的事务 id，只有子节点列表变更才会更新 pZxid，子节点内容变更不会更新
cversion:子节点版本号，当前节点的子节点每次变化时值增加 1
dataVersion:数据节点内容版本号，节点创建时为 0，每更新一次节点内容(不管内容有无变化)该版本号的值增加 1
aclVersion:节点的 ACL 版本号，表示该节点 ACL 信息变更次数
ephemeralOwner:创建该临时节点的会话的 sessionId；如果当前节点为持久节点，则 ephemeralOwner=0
dataLength:数据节点内容长度
numChildren:当前节点的子节点个数

### 2.3 ACL（权限控制）
对于 znode 的操作权限
* CREATE：支持创建子节点
* READ：能查询节点数据和列出子节点
* WRITE：能设置/更新节点数据
* DELETE：能删除子节点
* ADMIN：能设置节点 ACL 的权限

`CREATE 和 DELETE 都是对子节点的控制权限`
 对于身份认证，提供以下几种方式：
 * world：默认，any user can use
 * auth：任何已认证用户
 * digest： username：password
 * ip：对指定 IP 限制


 ### 2.4 Watcher（事件监听器）
 > 允许节点注册 Watch，在某些特定事件触发时，zk 服务端会将事件通知到感兴趣的客户端

![watche机制](../../images/watche机制.png)

### 2.5 Session（会话）
> 可以看作是 zk 与客户端之间的 TCP 长连接

Session：sessionTimeout 会话超时时间（范围内，仍属于同一个 session）
zk 与客户端创建 session 之前，会为客户端分配一个 sessionID，全局唯一

## 三、zk 集群
> 为了保证高可用，zk 需集群部署，只要集群中大部分机器可用，则集群可用。集群间通过 ZAB（zk Atomic Broadcase） 保证数据一致性
![zookeeper集群](../../images/zookeeper集群.png)

典型集群模式：Master/Slave（主从模式）

### 3.1 zk 集群中角色
> Leader,Follower,Observer
![zookeeper集群中的角色](../../images/zookeeper集群中的角色.png)
* Leader：提供读写，负责投票的发起和决议，更新系统状态
* Follower：提供读，参与选举投票，转发写请求给 Leader
* Observer：提供读，不参与 Leader 选举

**leader写数据**
![leader写数据](../../images/leader写数据.jpg)

**读数据**
![读数据](../../images/读数据.jpg)

当 Leader 服务器出现网络中断、崩溃退出与重启等异常情况，就会进入 Leader 选举过程
1. Leader election（选举阶段）：只要一个节点得到超过半数节点的票数，则可成为 Leader
2. Discovery（发现阶段）：Follower 与准 Leader 通信，同步 Follower 最近接收的事务提议
3. Synchronization（同步阶段）：利用 Leader 前一阶段获得的最新提议历史，同步集群所有的副本。同步完成准 Leader 成为 Leader
4. Broadcase（广播阶段）：zk 集群正式对外提供事务服务，Leader 可以进行消息广播。若有新节点加入，需对新节点进行同步

### 3.2 zk 集群中服务状态
* LOOKING：寻找 Leader
* LEADING：Leader 状态，对应的节点为 Leader
* FOLLOWING：Follower 状态，对应节点为 Follower
* OBSERVING：Observer 状态，对应节点为 Observer，不参与 Leader 选举

### 3.3 zk 集群数量
> zk 集群，服务器剩余数量大于宕机数量才能继续提供服务。即 2n 和 2n-1 的容忍度是一样的

### 3.4 zk 选举的过半机制防止脑裂
> 为保证 zk 集群可用性，服务器通常部署在不同机房，如果机房间网络线路故障，集群被割裂成几个小集群

过半机制防止脑裂：服务器少于等于一般不可能产生 Leader

## 四、ZAB 协议和 Paxos 算法
### 4.1 ZAB 协议简介
> ZAB（zk Atomic Broadcast 原子广播）协议是 zk 专门设计的一种支持崩溃恢复的原子广播协议。zk 中主要依赖 ZAB 协议来实现分布式数据一致性，基于该协议，zk 实现了一种主备模式的系统架构来保持集群中各个副本之间数据一致性

ZAB 协议两种基本的模式：崩溃恢复和消息广播
* 崩溃恢复：当服务框架在启动过程中，或是当 Leader 服务器出现网络中断、崩溃退出与重启等异常时，ZAB 协议就会进入恢复模式并选举新的 Leader。当选举产生了新的 Leader，切已经有过半的机器与 Leader 完成了状态同步后，ZAB 协议就会退出恢复模式。状态同步即数据同步，用来保证集群中存在过半的及其的数据与 Leader 一致
* 消息广播：当一台遵守 ZAB 协议的服务器启动后加入到集群中时，如果此时集群中已经存在 Leader 在负责进行消息广播，则新加入的服务器会自觉进入数据恢复模式，找到 Leader，进行数据同步

**选举算法：FastLeaderElection**
* myid：每个 zk 服务器在数据文件夹下创建名为 myid 的文件，内容全局唯一整数 ID

## 五、常用命令
1. help
2. create
3. set
4. get
5. ls
6. stat
7. ls2（ls + stat）
8. delete

## 六、总结
1. zk 是分布式程序（半数以上节点存活则 zk 能正常提供服务）
2. zk 数据保存在内存中
3. zk 是高性能
4. zk 中持久节点&临时节点
5. zk 底层提供两个功能：管理（存取）程序数据、为用户提供数据节点的监听


