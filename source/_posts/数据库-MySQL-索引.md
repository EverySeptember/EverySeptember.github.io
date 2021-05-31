---
title: MySQL调优-索引
date: 2019-05-29 18:48:17
tags: [数据库, MySQL, 索引]
---


索引是为表建立的快速访问的目录，避免全表扫描，MySQL中索引的存储形式是根据存储引擎决定的

# 分类

- 根据存储结构
  - BTree索引（B-Tree索引或者B+Tree索引）
  - Hash索引
  - full-index索引
  - R-Tree索引
- 根据应用层次
  - 普通索引
  - 唯一索引（效率最高）
  - 复合索引
- 根据物理顺序与键值的逻辑顺序关系
  - 聚集索引
  - 非聚集索引

# MySQL中常用的索引

- B+Tree索引 - 适用于范围查找
- Hash索引 - 适用于精准查找

# B+Tree索引

MySQL中InnoDB与MyISAM采用的索引

![B+Tree](https://ws3.sinaimg.cn/large/005BYqpggy1g3o5omzqquj30ic09u0te.jpg)

- InnoDB
  - B+Tree采用树形链表结构建立数据目录
  - B+Tree索引是一种聚集索引
  - 索引底层节点之间持有一个单项链表，更适合范围查找
  - 在其他字段上建立的索引会主动关联主键索引，然后根据主键去寻找数据
- MyISAM
  - B+Tree在MyISAM中是一种非聚集索引

其他：B-Tree同样是一棵树，但在底层节点之间没有联系，不适合范围查询

![B-Tree](https://ws3.sinaimg.cn/large/005BYqpggy1g3o5nwdaakj30fr09daai.jpg)

## 解释计划

使用explain+SQL语句，便可查看语句的执行情况，根据解释计划可对该SQL进行针对性的优化

## 使用方法

- 精确匹配与范围匹配都可以使用索引
- 模糊匹配（like）时前缀查询可以使用索引，效率与前缀筛选粒度有关
- 模糊匹配（like）时后缀查询不会使用索引，进行全表扫描
- 复合索引
  - 复合索引需要讲选择性高的索引放到左侧
  - 在使用复合索引的时候一定要包含左侧列，单使用右侧列不会使用索引
- <> 与 not in 不会使用索引
  - 对数字可使用大于或者小于的场景进行优化
- 查询范围太大也不会用索引

# Hash索引

- 基于哈希表实现
- 只有使用精准匹配所有列的时候才有效
- 为每一条数据创建哈希值，基于哈希值进行匹配查询
- 当前MySQL只有memory引擎才支持显式hash索引
  - Memory支持在创建表时对列创建hash所以你能

## 特点

- Hash索引只包含哈希值和行指针
- 只支持精准匹配，不支持范围查询、模糊查询及排序
- Hash取值非常快，但索引选择性很低时不建议使用

## InnoDB中的hash索引

- InnoDB存储引擎只支持显式创建BTree索引

- InnoDB精准匹配时会自动生成HashCode，存入缓存

# 索引的优缺点

- 优点
  - 大幅提高了数据的检索效率
  - BTree等聚集索引将随机IO整理成顺序IO，提高范围查询效率
- 缺点
  - 降低了写入数据的效率
  - 太多的索引增加了查询优化器的选择时间
  - 不合理的使用索引，会大幅占用磁盘空间

# 索引的优化

不会使用索引的情况

- 索引选择性太差
- <>或not in或is not null
- where子句跳过复合索引左侧索引列，直接使用右侧索引列
- 对索引列进行计算或函数处理（Oracle支持函数索引）的情况

对索引的分析

```mysql
SELECT
	object_type,		#对象类型
	object_schema,		#所属数据库
	object_name,		#对象名称
	index_name,			#索引名
	count_read,			#索引在计算过程中读取行数
	count_fetch,		#索引查询结果行数
	count_insert,		#通过索引进行新增操作的行数
	count_update,		#通过索引进行新增更新的行数
	count_delete 		#通过索引进行新增删除的行数
FROM
	performance_schema.table_io_waits_summary_by_index_usage 
ORDER BY
	sum_timer_wait DESC;
```

最后五列代表使用次数，当上线一段时间后，可用该语句进行分析，大部分使用次数都是0的索引可以删除。

# 使用索引优化排序

当order by字段与索引字段顺序、排序方向相同时，才可使用索引优化排序速度

- 当单字段使用索引时，升降序不会影响速度
- 当使用多列索引时，必须是升序，且顺序不允许打乱

# 减少表与索引碎片

- analyze table 表名
  - 对索引的统计信息进行重新计算
- optimize table 表名
  - 对表数据进行优化、重新组织

# 嵌套循环关联（Nested Loop Join）

MySQL中的执行流程

- 选择驱动表（MySQL执行）
  - 对SQL语句进行解释计划分析时，第一行出现的表就是驱动表
- 对驱动表进行检索
- 根据驱动表结果循环对关联表进行检索
- 输出结果

对关联查询的优化

- 判断驱动表，对驱动表加索引
- 对关联表外键添加索引