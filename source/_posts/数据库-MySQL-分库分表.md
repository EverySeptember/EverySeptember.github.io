---
title: MySQL调优-分库分表
date: 2019-7-23 21:55:13
tags: [数据库, MySQL]
---


# 分区表

分区表就是把大表按条件单独存储到不同的`物理小表`中，在构建出的完整`逻辑表`

## 优点

- 更少的数据检索范围
- 拆分超级大的表，将部分数据加载至内存
- 分区表的数据更容易维护
- 分区表数据文件可以分布在不同的硬盘上，并发IO
- 减少锁的范围，避免大表锁表
- 可独立备份、恢复分区数据

## 使用方法

`create table () partition by func(字段) (partition 分区名 values 分区条件...)` 

## 限制

- 查询必须包含分区列，不允许对分区列进行计算
 2. 分区列必须是数字类型
 3. 分区表不支持建立外键索引
 4. 建表时主键必须包含所有的列（影响主键索引性能）
 5. 最多1024个分区

# 分库分表

## 数据分库

将数据存放在多条MySQL服务器上

缺点：数据分布不均匀，未能根本解决海量数据存储问题

## 数据分表

将大表按照一定规则拆分成多个表，操作时对多表分别操作并进行合并

## 分库分表中间件（Sharding Sphere）

结构

![Sharding Sphere结构](https://s2.ax1x.com/2019/07/24/eESkB6.png)

### Sharding JDBC

- 定位为轻量级Java框架，在Java的JDBC层提供的额外服务
- 它使用客户端直接连接数据库，以jar包形式提供服务
- 可理解为增强版的JDBC驱动，完全兼容JDBC和各种ORM框架（mybatis、hibernate等）

### 配置

applicationContext.xml

```xml
<sharding:data-source id="shardingDataSource">
        <!--
            sharding:sharding-rule定义分表规则:
            data-source-names="ds0,ds1" 说明有几个数据源
            sharding:table-rule 节点定义数据存储规则
            logic-table="t_order" 代表逻辑表名
            actual-data-nodes="ds$->{0..1}.t_order_$->{0..1}"
            说明数据分布在ds0与ds1两个数据源,物理表存在t_order_0与t_order_1
            database-strategy-ref="databaseStrategy" 说明order表分库策略
            基于order.user_id值对2取余保存到ds0或ds1的数据库中
            table-strategy-ref="orderTableStrategy"
            基于order.order_id对2取余决定存储在对应数据库的 t_order_0或者t_order_1表中
            实例:

            insert into t_order(order_id , user_id,status) values( ? , ? , 'N')
            order_id  user_id
            1         2           -> ds0.order_1
            2         2           -> ds0.order_0
            3         1           -> ds1.order_1
            4         1           -> ds1.order_0
        -->
	...        
</sharding:data-source>
```

### 分布式主键

在分库分表后，不要使用数据库自动生成主键，需采用[分布式主键](https://shardingsphere.apache.org/document/current/cn/features/sharding/other-features/key-generator/)生成策略

## 读写分离

1. 配置主从数据库

2. 配置applicationContext.xml中Sharding Sphere的读写分离

   ```xml
   <!--
       定义ShardingJDBC主从分离(读写)数据源
       ds0是主服务器,用于写入操作
       ds1是从属服务器,用于读取操作
       其中,ds0也承担一部分查询职责.(生产环境不建议这样做)
   -->
   <master-slave:data-source id="msDataSouce"
                             master-data-source-name="ds0"
                             slave-data-source-names="ds0,ds1">
       <master-slave:props>
           <prop key="sql.show">true</prop>
       </master-slave:props>
   </master-slave:data-source>
   ```

# 一主多从数据同步

不同服务器上数据的完全同步

原理：从服务器从主服务器上读取操作日志，并执行其中的SQL语句

1. 修改主从服务器的my.ini文件

   - server-id不能相同

2. 在主服务器上为从属服务器创建用户，并赋予其权限

3. 查询主服务器日志状态

   ```sql
   show master status
   ```

4. 在从属服务器上执行

   ```sql
   CHANGE MASTER TO 
   MASTER_HOST='主服务器IP',
   MASTER_PORT=主服务器端口,
   MASTER_USER='主服务器开放用户',
   MASTER_PASSWORD='主服务器开放用户密码',
   MASTER_LOG_FILE='主服务器日志文件',
   MASTER_LOG_POS=主服务器查询到的最后位置;
   ```

5. 从属服务器开始执行主从复制

   ```sql
   start slave
   ```

