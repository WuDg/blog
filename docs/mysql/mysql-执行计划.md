# MySQL explain
---

> explain关键字可以模拟MySQL优化器执行SQL语句，可以很好地分析SQL语句或表结构的性能瓶颈

2021年3月5日17:33:33

## explain作用

1. 表的读取顺序如何
2. 数据读取操作有哪些操作类型
3. 哪些索引可以使用
4. 哪些索引被实际使用
5. 表之间是如何引用
6. 每张表有多少行被优化器查询

...

## explain的执行效果

```mysql
mysql> explain select * from tb_user where id = 1 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_user
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
        Extra: NULL
1 row in set (0.00 sec)
```



## explain包含的字段

1. id	// select查询的序列号，包含一组数字，表示查询中执行select字句或操作表的顺序
2. select_type    // 查询类型
3. table    // 正在访问哪个表
4. partitions    // 匹配的分区
5. type    // 访问的类型
6. possible_keys    // 显示可能应用在这张表中的索引，一个或多个，但不一定实际使用到
7. key    // 实际使用到的索引，如果为NULL，则没有使用索引
8. key_len    // 表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度，值越大效率越高
9. ref    // 显示索引的哪一列被使用了，如果可能的话，是一个常数，哪些列或常量被用于查找索引上的值
10. rows    // 根据表统计信息及索引选用情况，大致估算出找到所需的记录所需读取的行数
11. filtered    // 查询的表行占表的百分比   
12. Extra    // 包含不适合在其他列中显示但十分重要的额外信息



## 详细内容

### id字段

**1. id相同**

```mysql
explain select * from tb_subject sub,tb_user us,tb_user_subject usu where sub.id=usu.subject_id and us.id=usu.user_id and us.id=1;
```



**2. id不同**

`如果是子查询，id的序号会递增，id的值越大优先级越高，越先被执行`

读取顺序：tb_user > tb_user_subject

```mysql
explain select subject_id from tb_user_subject usu where usu.user_id= (select id from tb_user us where id=1);
```



**3. id相同又不同**

`id如果相同，则从上往下顺序执行。id不同，则id值越大，越先执行`

```mysql
explain select * from tb_subject sub LEFT JOIN tb_user_subject usu on sub.id=usu.subject_id where usu.user_id=1 union all select * from tb_subject sub LEFT JOIN tb_user_subject usu on sub.id=usu.subject_id where usu.user_id=1;
```


### select_type字段

**1. SIMPLE**

`简单查询，不包含子查询或union查询`

```mysql
explain select * from tb_subject sub,tb_user us,tb_user_subject usu where sub.id=usu.subject_id and us.id=usu.user_id and us.id=1;
```


**2. PRIMARY**

`查询中若包含任何复杂的子查询，最外层查询则被标记为主查询PRIMARY`

```mysql
explain select subject_id from tb_user_subject usu where usu.user_id= (select id from tb_user us where id=1)
```


**3. SUBQUERY**

`在select或where中包含子查询`

explain select subject_id from tb_user_subject usu where usu.user_id= (select id from tb_user us where id=1)



**4. DERIVED**

`在from列表中包含的子查询被标记为DERIVED`

```mysql
explain select name from (select * from tb_user us where id=1) tmp;
```


**5. UNION**

`若第二个select出现在union之后，则被标记为UNION`

```mysql
explain select * from tb_user where id=1 union select * from tb_user where id=2;
```



**6. UNION RESULT**

`从UNION表获取结果的select`

```mysql
explain select * from tb_user where id=1 union select * from tb_user where id=2;
```


### type字段

```properties
NULL>system>const>eq_ref>ref>fulltext>ref_or_null>index_merge>unique_subquery>index_subquery>range>index>ALL //最好到最差
```



**备注：掌握以下10中常见的即可**

```properties
NULL>system>const>eq_ref>ref>ref_or_null>index_merge>range>index>ALL
```

**1. NLL**

`MySQL能够在优化阶段分解查询语句，在执行阶段不用再访问表或索引`

```mysql
explain select min(id) from tb_user;
```

**2. system**

`表中只有一行记录，这是const类型的特例，平时不太会出现，可以忽略`

没有实例

**3. const**

`表示通过索引一次就找到了，const用于比较primary key或unique索引，`

```mysql
explain select * from tb_system where id=1;
```


**4. eq_ref**

`唯一性索引扫描，对于每个索引键，表中只有一条记录与之对应，常见于主键或唯一索引扫描`

```mysql
explain select subject_id from tb_user_subject usu LEFT JOIN tb_subject su on usu.subject_id=su.id;
```


**5. ref**

`非唯一性索引扫描，返回匹配某个单独值得所有行`

```mysql
explain select subject_id from tb_subject su LEFT JOIN tb_user_subject usu on usu.subject_id=su.id;
```

**6. ref_or_null**

`类似ref，但是可以搜索值为NULL的行`

```mysql
explain select * from tb_user_subject usu where subject_id=1 or subject_id is null;
```


**7. index_merge**

`表示使用了索引合并的优化方法`

```mysql
explain select * from tb_user_subject where id = 1 or subject_id='2';
```


**8. range**

`只检索给定范围的行，使用一个索引来选择行`

```mysql
explain select * from tb_user where id BETWEEN 1 and 3;
```


**9. index**

`Full index Scan，Index与All区别：index只遍历索引树，通常比All快，因为索引文件通常比数据文件消息，也就是虽然all和index都是读全表，但index是从索引中读的，而all是从硬盘读的`

```mysql
explain select id from tb_user;
```

**10. ALL**

`Full Table Scan，将遍历全表以找到匹配行`

```mysql
explain select * from tb_user;
```


### table字段

**数据来自哪张表**



### possible_keys字段

**显示可用应用在这张表中的索引，一个或多个，查询涉及到的字段若润在索引，则该索引将被列出，但不一定被实际使用**

### key字段

**实际使用到的索引，如果为NULL，则没有使用索引，查询中若使用了覆盖索引（查询的列刚好是索引），则该索引进出现在key列表**



### key_len字段

**表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度，在不损失精确度的情况下，长度越长越好，key_len显示的值为索引字段最大的可能长度，并非实际使用长度，即key_len是根据定义计算而得，不是通过表内索引出**



### ref字段

**显示索引的哪一列被使用了，如果可能的话，是一个常数，哪些列或常量被用于查找索引列上的值**



### row字段

**根据表统计信息及索引选用情况，大致估算出找到所需的记录所需读取的行数**



### partitions字段

**匹配的分区**



### filtered字段

**查询的表行占表的百分比**



### Extra字段

**包含不适合在其他列显示但十分重要的额外信息**



**1. Using filesort**

**说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取，MySQL中无法利用索引完成的排序操作成为 [文件排序]**

```mysql
explain select * from tb_user order by name;
```


**2. Using temporary**

`使用了临时表保存中间结果，MySQL在对结果排序时使用临时表，常见于排序order by和分组查询group by`

```mysql
explain select subject_id from tb_user_subject usu where usu.user_id= (select id from tb_user us where id=1) union all select subject_id from tb_user_subject usu where usu.user_id= (select id from tb_user us where id=1);
```


**3. Using index**

`表示相应的select操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，`

```mysql
explain select subject_id from tb_user_subject usu where usu.user_id= (select id from tb_user us where id=1);
```


**4. Using where**

`使用了where条件`


**5. Using join buffer**

`使用了连接缓存`

```mysql
explain select * from tb_subject sub,tb_user us,tb_user_subject usu;
```


**6. impossible where**

`where自居的值总是false，不能用来获取任何元组`

```mysql
explain select * from tb_user where id=1 and id=3;
```


**7. distinct**

`一旦MySQL找到了与行相联合匹配的行，就不再搜索了`

```mysql
explain select distinct(name) from tb_user us left join tb_user_subject sub on sub.user_id=us.id;
```

**8. Select tables optimized away**

`select操作已经优化到不能再优化了，MySQL没有遍历表或索引就返回数据了`

```mysql
explain select min(id) from tb_user;
```


**使用的数据表**

```mysql
/*
Navicat MySQL Data Transfer

Source Server         : loca
Source Server Version : 50624
Source Host           : localhost:3306
Source Database       : explain-learn

Target Server Type    : MYSQL
Target Server Version : 50624
File Encoding         : 65001

Date: 2021-03-05 17:30:25
*/

SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for tb_subject
-- ----------------------------
DROP TABLE IF EXISTS `tb_subject`;
CREATE TABLE `tb_subject` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `subject_name` varchar(255) DEFAULT NULL COMMENT '学科名',
  `subject_teacher` varchar(255) DEFAULT NULL COMMENT '学科老师姓名',
  `subject_code` varchar(255) DEFAULT NULL COMMENT '学科代码',
  PRIMARY KEY (`id`),
  KEY `subject_code` (`subject_code`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of tb_subject
-- ----------------------------
INSERT INTO `tb_subject` VALUES ('1', '语文', '张三', '1000');
INSERT INTO `tb_subject` VALUES ('2', '数学', '李四', '2000');
INSERT INTO `tb_subject` VALUES ('3', '英语', '王五', '3000');
INSERT INTO `tb_subject` VALUES ('4', '物理', '赵六', '4000');
INSERT INTO `tb_subject` VALUES ('5', '化学', '田七', '5000');
INSERT INTO `tb_subject` VALUES ('6', '生物', '李八', '6000');

-- ----------------------------
-- Table structure for tb_system
-- ----------------------------
DROP TABLE IF EXISTS `tb_system`;
CREATE TABLE `tb_system` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `desc` varchar(255) DEFAULT NULL COMMENT '描述',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of tb_system
-- ----------------------------
INSERT INTO `tb_system` VALUES ('1', '我只有一行');

-- ----------------------------
-- Table structure for tb_user
-- ----------------------------
DROP TABLE IF EXISTS `tb_user`;
CREATE TABLE `tb_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '用户id',
  `name` varchar(255) DEFAULT NULL COMMENT '用户名',
  `age` int(11) DEFAULT NULL COMMENT '年龄',
  `user_no` varchar(11) DEFAULT NULL COMMENT '工号，唯一',
  PRIMARY KEY (`id`),
  UNIQUE KEY `user_no` (`user_no`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of tb_user
-- ----------------------------
INSERT INTO `tb_user` VALUES ('1', 'henry', '18', '0001');
INSERT INTO `tb_user` VALUES ('2', 'lucy', '20', '0002');
INSERT INTO `tb_user` VALUES ('3', 'jack', '66', '0003');

-- ----------------------------
-- Table structure for tb_user_subject
-- ----------------------------
DROP TABLE IF EXISTS `tb_user_subject`;
CREATE TABLE `tb_user_subject` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) DEFAULT NULL,
  `subject_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `subject_id` (`subject_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of tb_user_subject
-- ----------------------------
INSERT INTO `tb_user_subject` VALUES ('1', '1', '1');
INSERT INTO `tb_user_subject` VALUES ('2', '2', '1');
INSERT INTO `tb_user_subject` VALUES ('3', '3', '3');
INSERT INTO `tb_user_subject` VALUES ('4', '1', '4');
INSERT INTO `tb_user_subject` VALUES ('5', '2', '4');
INSERT INTO `tb_user_subject` VALUES ('6', '3', '6');
INSERT INTO `tb_user_subject` VALUES ('7', '1', '6');
INSERT INTO `tb_user_subject` VALUES ('8', '2', '5');
INSERT INTO `tb_user_subject` VALUES ('9', '3', '4');
INSERT INTO `tb_user_subject` VALUES ('10', '1', '3');
INSERT INTO `tb_user_subject` VALUES ('11', '2', '2');
INSERT INTO `tb_user_subject` VALUES ('12', '3', '1');

```