# MySQL-锁

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/mysql.jpg)
---

## 一、锁的认识

**1.1 锁的解释**

计算机协调多个进程或线程并发访问某一资源的机制

**1.2锁的重要性**

在数据库中，除传统计算资源（CPU、RAM、I\O等）的争抢，数据也是一种供多用户共享的资源。如何保证数据并发访问的一致性、有效性，是所有数据库必须要解决的问题。

锁冲突也是影响数据库并发访问性能的一个重要因素，因此锁对数据库尤其重要

**1.3 锁的缺点**

加锁是消耗资源的，锁的各种操作，包括获得锁、检测锁是否已解除、释放锁等，都会增加系统的开销

**1.4 例子**

网购，商品库存有限，保证商品不会超卖，用户下单后系统会该商品进行锁定，防止其他用户重复下单，直到支付动作完成才会释放

## 二、MySQL锁的类型

### 2.1 表锁

**读锁（read lock）**：也叫共享锁（shared lock）

针对同一份数据，多个读操作可以同时进行而不会互相影响（select）

**写锁（write lock）**：也叫排他锁（exclusive lock）

当前操作没完成之前，也会阻塞其他读和写操作（update、insert、delete）

**MyISAM存储引擎**默认为表锁

特点：
1. 对整张表加锁
2. 开销小
3. 加锁快
4. 无死锁
5. 锁粒度大，发生锁冲突概率大，并发性低

**总结**
1. 读锁会阻塞写操作，不会阻塞读操作
2. 写锁会阻塞读和写操作

**建议**

MyISAM的读写锁调度是写优先，这也是MyISAM不适合做写为主表的引擎，因为写锁以后，其他线程不能做任何操作，大量的更新使查询很难得到锁，从而造成永远阻塞

### 2.2 行锁

**读锁（read lock）**：也叫共享锁（shared lock）

允许一个事务去读一行，组织其他事务获得相同数据集的排他锁

**写锁（write lock）**：也叫排他锁（exclusive lock）

允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的共享锁和排他锁

**意向锁（IS）**

一个事务给一个数据行加共享锁时，必须先获得表的IS锁

**意向排他锁（IX）**

一个事务给一个数据行加排他锁时，必须鲜活的该表的IX锁

**InnoDB默认的锁为行锁**

**特点：**
1. 对一行数据加锁
2. 开销大
3. 加锁慢
4. 会出现思索
5. 锁粒度小，发生锁冲突概率最低，并发性高

**事务并发带来的问题**

1. 更新丢失。解决：让事务变成串型操作，而不是并发的操作，即对每个事务开始一一对读取记录加排他锁
2. 脏读。解决：隔离级别为Read uncommitted
3. 不可重复读。解决：使用Next-key Lock算法来避免
4. 幻读。解决：间隙锁（Gap Lock）

### 2.3 页锁
> 开销、加锁时间和锁粒度介于表锁和行锁之间，会出现思索，并发处理能够力一般

## 三、上锁
### 3.1 行锁

**隐式上锁**
```sql
select // 上读锁
```

```sql
// 上写锁
insert
update
delete
```

**显示上锁（手动）**

```sql
lock table tablename read; // 读锁

lock table tablename write; //写锁
```

**解锁（手动）**

```sql
unlock tables; // 所有锁表
```

| session01                                                    | session02                                               |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| lock table teacher read;// 上读锁                            |                                                         |
| select * from teacher; // 可以正常读取                       | select * from teacher;// 可以正常读取                   |
| update teacher set name = 3 where id =2;// 报错因被上读锁不能写操作 | update teacher set name = 3 where id =2;// 被阻塞       |
| unlock tables;// 解锁                                        |                                                         |
|                                                              | update teacher set name = 3 where id =2;// 更新操作成功 |

| session01                                                   | session02                                               |
| ----------------------------------------------------------- | ------------------------------------------------------- |
| lock table teacher write;// 上写锁                          |                                                         |
| select * from teacher; // 可以正常读取                      | select * from teacher;// 被阻塞                         |
| update teacher set name = 3 where id =2;// 可以正常更新操作 | update teacher set name = 4 where id =2;// 被阻塞       |
| unlock tables;// 解锁                                       |                                                         |
|                                                             | select * from teacher;// 读取成功                       |
|                                                             | update teacher set name = 4 where id =2;// 更新操作成功 |

### 3.2 行锁

**隐式上锁（默认，自动加锁自动释放）**

```sql
select // 不会上锁
```

```sql
// 上写锁
insert
update
delete
```

**显示上锁（手动）**

```sql
select * from tableName lock in share mode; // 读锁
```

```sql
select * from tableName for update; // 写锁
```



**解锁（手动）**

1. 提交事务（commit）
2. 回滚事务（rollback）
3. kill阻塞进程



| session01                                                    | session02                                               |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| begin;                                                       |                                                         |
| select * from teacher where id = 2 lock in share mode;// 上读锁 |                                                         |
|                                                              | select * from teacher where id = 2;// 可以正常读取      |
| update teacher set name = 3 where id =2;// 可以更新操作      | update teacher set name = 5 where id =2;// 被阻塞       |
| commit;                                                      |                                                         |
|                                                              | update teacher set name = 5 where id =2;// 更新操作成功 |

| session01                                               | session02                                               |
| ------------------------------------------------------- | ------------------------------------------------------- |
| begin;                                                  |                                                         |
| select * from teacher where id = 2 for update;// 上写锁 |                                                         |
|                                                         | select * from teacher where id = 2;// 可以正常读取      |
| update teacher set name = 3 where id =2;// 可以更新操作 | update teacher set name = 5 where id =2;// 被阻塞       |
| rollback;                                               |                                                         |
|                                                         | update teacher set name = 5 where id =2;// 更新操作成功 |

**问**：为什么上了写锁，别的事务还可以读操作？

**答**：因为InnoDB有MVCC机制（多版本并发控制），可以使用快照读，而不会被阻塞

## 四、行锁的实现算法

### 4.1 Record Lock锁

单个行记录上的锁

Record Lock总是会去锁住索引记录，如果InnoDB存储引擎表简历的时候没有设置任何索引，这是InnoDB存储引擎会使用隐式的主键来进行锁定



### 4.2 Gap Lock锁

当我们使用范围条件而非相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引加锁

**优点**：解决了事务并发的幻读问题

**不足**：因为query执行过程中通过范围查找的话，会锁定整个范围内的所有的索引键值，即使这个键值并不存在

**间隙锁缺点**：当锁定一个范围键值之后，即使某些不存在的键值也会被无故的锁定，而造成锁定的时候无法插入锁定键值范围内任何数据



### 4.3 Next-key Lock锁

`同时锁住数据+间隙锁`

在Repeatable Read隔离级别下，Next-key Lock算法是默认的行记录锁定算法



**行锁注意**：

1. 只有通过索引条件检索数据时，InnoDB才会使用行级锁，否则会使用表级锁（索引失效，行锁边表锁）
2. 即使是访问不同行的记录，如果使用的是相同的索引键，会发生锁冲突
3. 如果数据表建有多个索引，可以通过不同的索引锁定不同的行

## 五、排查锁

### 5.1 表锁

**查看表锁情况**

`本文收录于:`
* [Github仓库:wudg/blog](https://github.com/wudg/blog)
* [Gitee仓库:wudg/blog](https://githee.com/wudg/blog)
