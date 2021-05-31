---
title: MySQL调优-工具
date: 2019-6-9 15:00:11
tags: [数据库, MySQL, 慢SQL日志, 执行计划]
---

# 慢SQL日志

- 开启慢SQL

  ```mysql
  set global slow_query_log = on;
  ```

- 设置慢SQL执行时间阈值，以秒为单位

  ```mysql
  set global long_query_time = 0.001;
  ```

- 指定慢SQL日志文件文件，目录保存在mysql/data下

  ```mysql
  set global slow_query_file = "文件名";
  ```

- 是否记录没有使用索引的SQL语句（系统表查询也会被记录，冗余数据无法排出 ）

  ```mysql
  set global log_queries_not_using_indexes = on;
  ```

- 永久生效可将上述语句保存在my.cnf配置文件中

# 执行计划

使用explain关键字加SQL语句；

## 执行计划列含义

- id列，编号是 select 的序列号，有几个 select 就有几个id，并且id的顺序是按 select 出现的顺序增长的。当多表关联时，id=1的表是驱动表

- **select type**列，查询类型,说明查询的种类

  - simple：简单查询。查询不包含子查询和union
  - primary：复杂查询中最外层的 select
  - derived：包含在 from 子句中的子查询。MySQL会将结果存放在一个临时表中，也称为派生表（derived的英文含义）
  - union：在 union 中的第二个和随后的 select，union操作会产生union result，其结果使用临时表存储的，没有索引，应尽量减少使用
  - union result：从 union 临时表检索结果的 select
  - subquery：包含在 select 中的子查询（不在 from 子句中）

- table列，表示执行计划的当前行正在访问哪个表

- partitions列，说明查询作用在哪个分区表上

- **type**列，这一列表示关联类型或访问类型，即MySQL决定如何查找表中的行。type包含以下12种值，其效率依次递减（加粗为常用类型）

  1. system

  2. **const**，mysql能对查询的某部分进行优化并将其转化成一个常量。用于 primary key 或 unique key 的所有列与常数比较时，所以表最多有一个匹配行，读取1次，速度比较快。

     `explain select * from (select * from film where id = 1) tmp;`

  3. **eq_ref**，primary key 或 unique key 索引的所有部分被连接使用 ，最多只会返回一条符合条件的记录。这可能是在 const 之外最好的联接类型了，简单的 select 查询不会出现这种 type。（多出现于多表关联时）

     `explain select * from film_actor left join film on film_actor.film_id = film.id;`

  4. **ref**，相比eq_ref，不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行

     `explain select * from film where name = "film1";`

  5. fulltext，全文索引

  6. **ref_or_null**，类似ref，但是可以搜索值为NULL的行

     `explain select * from film where name = "film1" or name is null;`

  7. index_merge

  8. unique_subquery

  9. index_subquery

  10. **range**，范围扫描，通常出现在 in(), between ,> ,<, >= 等操作中。使用一个索引来检索给定范围的行

  11. **index**，和ALL一样，不同就是mysql只需扫描索引树，这通常比ALL快一些

      `explain select count(*) from film;`

  12. **ALL**，即全表扫描，意味着mysql需要从头到尾去查找所需要的行。通常情况下这需要增加索引来进行优化了

      `explain select * from actor;`

- possible_keys列，这一列显示查询可能使用哪些索引来查找

- key列，这一列显示mysql实际采用哪个索引来优化对该表的访问

- key_len列，这一列显示了mysql在索引里使用的字节数，通过这个值可以算出具体使用了索引中的哪些列

- ref列，这一列显示了在key列记录的索引中，表查找值所用到的列或常量，常见的有：const（常量），func，NULL，字段名（例：film.id）

- row列，这一列是mysql估计要读取并检测的行数，注意这个不是结果集里的行数

- filtered列，是一个百分比的值，代表 (rows * filtered) / 100，这个结果将于前表产生交互，是一个预估值

- Extra列，这一列展示的是额外信息，以下是可能产生的信息

  - distinct：一旦mysql找到了与行相联合匹配的行，就不再搜索了

    `explain select distinct name from film left join film_actor on film.id = film_actor.film_id;`

  - **Using index**：这发生在对表的请求列都是同一索引的部分的时候，返回的列数据只使用了索引中的信息，而没有再去访问表中的行记录。是性能高的表现。也叫做索引覆盖。

    `explain select id,name from film order by id;`

  - Using where：mysql服务器将在存储引擎检索行后再进行过滤。就是先读取整行数据，再按 where 条件进行检查，符合就留下，不符合就丢弃。

    `explain select * from film where id > 1;`

  - using temporary：mysql需要创建一张临时表来处理查询。出现这种情况一般是要进行优化的，首先是想到用索引来优化

    `explain select distinct name from actor;（name字段无索引时）`

  - using filesort：采用文件扫描对结果进行计算排序，效率很差

    对于排序，只有select字段与order by字段都被索引覆盖是才允许使用Using Index

    `explain select name from actor order by name;（name字段需要有索引）`