---
title: MySQL调优-存储引擎
date: 2019-5-26 16:10:37
tags: [数据库, MySQL, MySQL存储引擎]
---

# MySQL体系结构

![MySQL体系结构](https://ws3.sinaimg.cn/large/005BYqpggy1g3esol5eb5j30jc0dwwhd.jpg)

# 存储引擎

存储引擎只针对表，每个表只允许有一个存储引擎，数据库中允许出现多个不同引擎

常见的有InnoDB、MyISAM、Memory等

创建表时选择引擎：`CREATE TABLE 表名() ENGINE 引擎名;`

## InnoDB

1. MySQL5.5.8后默认的存储引擎
2. InnoDB采用“表空间”保存文件
3. InnoDB支持事务处理

### 存储特性

- 表空间的两种形式
  - 早期使用系统表空间ibdata1（ibdataN）中，所有数据放到一个文件中
  - 5.6之后默认使用独立表空间：表明.ibd（推荐）
- 设置innodb_file_per_table修改表空间模式
- 支持部分的内存存储

### 事务特性

- InnoDB支持事务
- InnoDB默认使用行级锁
- InnoDB具有良好的高并发特性

#### MySQL的锁

- 职责分类
  - 共享锁-读锁
  - 独占锁（排它锁）-写锁
- 粒度分类
  - 行级锁
  - 表级锁（包括读取同样被锁）

InnoDB中只有利用索引的写操作才是行级锁，不能使用索引的写操作时表级锁

### 适用场景

- MySQL5.7开始有InnoDB支持全文索引与空间函数
- 优先推荐使用

## MyISAM

- 不支持事务
- 支持全文检索，支持text支持前缀索引
- 支持数据压缩
- 紧密存储，顺序读性能良好
- 表级锁，混合读写性能不加，并发性差

#### 适用场景

- 非事务应用，如保存日志
- 只读类应用，如报表数据、字典数据
- 空间类应用（5.7之前）
- MySQL自动创建的系统临时表，SQL查询，分组的临时表引擎

## Memory

- 不支持事务
- 内存读写，临时存储
- 超高的读写效率，比MyISAM高一个量级
- 表级锁，并发性差

#### 适用场景

- 读多写少的静态数据
- 充当缓存使用，保存高频访问静态数据
- MySQL自动创建的系统临时表

#### 关键参数

- max_heap_table_size：控制内存表大小（字节）
  - 默认16m
- tmp_table_size：设置内存临时表最大值（字节）

## CSV

- 纯文本保存
- 不支持事务
- 不支持索引
- 不允许空列

#### 适用场景

- 数据交换/数据迁移
- 不依赖MySQL环境

