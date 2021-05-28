---
title: Redis
date: 2019-7-29 21:06:05
tags: [NoSQL, Redis]
---

# Redis

Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

## 常用基本配置项

默认配置文件：根目录下redis.conf文件；

常用redis基本配置项：

```conf
# 是否后台运行，共两个值yes|no，默认值为no
daemonize yes
# 启动端口，默认值为6379
port 6379
# 指定日志文件地址
logfile ./redis.log
# 数据库总量
databases 256
# 数据文件存放目录
dir ./data/
# 密码
requirepass 123456
```

## 通用指令

### 启停

- 启动服务：`./src/redis-server [自定义配置文件地址]`
- 客户端连接：`./src/redis-cli [-h host] [-p port] [-a password]` 默认连接本地6379端口
- 服务器关闭：`./src/redis-cli shutdown [save|nosave]`关闭时是否持久化当前数据，默认nosave

### 通用指令

- 选择数据库：`select 数据库号`
- 存入一组值：`set key value`
- 查询一组值：`get key`，返回值为当前key对应的value
- 根据正则表达式查询复合条件的key：`keys pattern`，[支持的正则表达模式](http://www.redis.cn/commands/keys.html)；是一个阻塞的指令，不要在生产环境使用
- 查询当前数据库有多少数据：`dbsize`
- 查询目标key是否存在：`exists key`，返回值1存在，0不存在
- 删除一条数据：`del key`
- 对目标key设置有效期：`expire key seconds`
- 查询目标key的剩余有效时间：`ttl key`

### 其他指令

- 清空当前数据库：`flushdb`
- 清空所有数据库：`flushall`

## 数据类型

### Keys

Redis key值是二进制安全的，这意味着可以用任何二进制序列作为key值，从形如”foo”的简单字符串到一个JPEG文件的内容都可以。空字符串也是有效key值。

### value

以下为value支持的五种数据类型及其在redis内部存储的编码结构

- 字符串类型`String`
  - int 编码：保存long 型的64位有符号整数
  - embstr 编码：保存长度小于44字节的字符串
  - raw 编码：保存长度大于44字节的字符串
- Hash类型`Hash`
- 列表类型`List`
- 集合类型`Set`
- 有序集合类型`ZSet`

### String

String最大支持512M，实际生产中不建议存储超过100kb

#### 字符串常用指令

- 一次设置多个K-V：`mset [key1 value1] [key2 value2] [...]`
- 一次取出多个值：`mget [key1] [key2] [...]`
- 目标值自增/自减1：`incr/decr key`
- 目标值自增/自减任意值：`incrby/decrby key num`

### Hash

适合存储结构化数据

#### Hash指令

- 设置单个值：`hset key filed value`
- 获取单个值：`hget key filed`
- 设置/获取多个值：`hmset key filed value [filed value]` / `hmget key [key]`
- 获取所有值：`hgetall key`；返回值：单数行为filed，偶数行为对应的value
- 删除目标值：`hdel key filed`
- 检查是否存在：`hexists key filed`；返回值：1存在，0不存在
- 目标key中字段的数量：`hlen key`

#### 与String的存储优劣对比

| 命令              | 优点                     | 缺点                                       |
| :---------------- | :----------------------- | :----------------------------------------- |
| String-JSON序列化 | 编程简单、节约内存       | 需要额外的序列化开销、无法实现部分属性更新 |
| Hash              | 直观、可实现部分属性更新 | 多层嵌套困难、序列化/反序列化编程麻烦      |

### List

列表是简单的字符串列表，按照插入顺序排序

#### List指令

- 在列表左/右侧插入元素：`lpush/rpush key value [value ...]`
- 从列表的左/右侧弹出元素：`lpop/rpop key`
- 获取列表长度：`llen key`
- 获取列表子集：`lrange startpoint endpoint`；当endpoint为负值时，从右侧倒序查找位置

### Set

Redis中的Set是字符串类型的无序且唯一的集合。Set是通过哈希表实现的，所以添加、查找、删除的速度都是很快的。

#### Set指令

- 新增/移除元素：`sadd/srem key element`
- 随机获取/弹出元素：`srandmemeber/spop key [count]`
- 查看所有元素：`smembers key`
- 计算元素的数量：`scard key`
- 集合计算
  - 差集：`sdiff key [key]`
  - 交集：`sinter key [key]`
  - 并集：`sunion key [key]`

### ZSet

ZSet是字符串类型的有序且唯一的集合。在ZSet中，除了元素之外，还有一个分数值，ZSet会按照分数值对元素进行从小到大才排序。

#### ZSet指令

- 添加一个元素：`zadd key score member [score member...]`
- 移除一个元素：`zrem key member`
- 查看一个元素的分数：`zscore key member`
- 查看元素的在集合中的位置：`zrank key member`
- 查看集合中的元素数量：`zcard key`
- 查看集合中的元素子集：`zrange key start end [withscores]`；使用可选参数withscores同时查询分数
- 查看目标分数区间内的元素数量：`zcount key min max`
- 查看目标分数区间内的元素详细信息：`zrangebyscore key min max [withscores]`

## 客户端

首先需要打开远程访问权限，在conf文件中添加配置

```
bind 允许远程访问的host地址
```

### Jedis

Jedis是Java语言开发的Redis客户端开发工具包，对Redis指令进行了简单的封装。

添加引用：

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.1.0</version>
</dependency>
```

新建Jedis连接：

  ```java
Jedis jedis = new Jedis("host", "port");
  ```

#### Spring使用JedisPool连接池

##### Spring-Data-Redis配置选项

Maven引用：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-redis</artifactId>
        <version>2.1.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.9.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.1.8.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.8</version>
    </dependency>
    <dependency>
        <groupId>commons-beanutils</groupId>
        <artifactId>commons-beanutils</artifactId>
        <version>1.9.3</version>
    </dependency>
</dependencies>
```

applicationContext.xml：

```xml
<!-- 连接池配置类-->
<bean id="poolConfig"
      class="redis.clients.jedis.JedisPoolConfig"
      p:maxTotal="100"
      p:maxIdle="50"
      p:minIdle="10"
      p:maxWaitMillis="1000"
      p:blockWhenExhausted="true"
      p:testOnBorrow="false"
      p:testOnReturn="false"
      p:testOnCreate="true"
>
</bean>

<!-- Jedis连接配置 -->
<bean id="jedisConnFactory"
      class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
      p:usePool="true"
      p:hostName="39.104.126.40"
      p:port="6379"
      p:password="123456"
      p:database="0"
      p:poolConfig-ref="poolConfig"
>
</bean>

<!-- RedisTemplate配置 -->
<bean id="redisTemplate"
      class="org.springframework.data.redis.core.RedisTemplate"
      p:connectionFactory-ref="jedisConnFactory"
    >
    <!-- SpringDataRedis默认采用JDK二进制序列化工具 -->
    <!-- 采用以下设置，将key和value分别用String方式进行序列化，提高Redis中的数据可读性 -->
    <property name="keySerializer">
        <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"></bean>
    </property>
    <property name="valueSerializer">
        <bean class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer"></bean>
    </property>
    
    <property name="hashKeySerializer">
        <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"></bean>
    </property>
    <property name="hashValueSerializer">
        <bean class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer"></bean>
    </property>
</bean>
```

##### RedisTemplate模板对象

- Spring使用RedisTemplate来进行Redis操作
- 使用`RedisTemplate.opsFor*()`函数来获取各Redis数据类型的操作对象
- 注入好RedisTemplate后不再需要管理Jedis的开启与销毁，可直接使用

##### Java对象在Redis中的序列化与反序列化

- SpringDataRedis默认采用JDK二进制序列化工具，在开发过程中可能会影响可读性
- 分别配置`*KeySerializer`和`*ValueSerializer`实现对Key和Value的序列化
- 当希望将对象以Json的形式存储的时候，SpringDataRedis底层默认使用Jackson，需要添加引用并使用`GenericJackson2JsonRedisSerializer`类进行序列化与反序列化

### 声明式缓存

使用注解的方式对当前应用进行“非侵入式”缓存扩展

#### SpringCache

SpringCache是Spring生态中的一员，用于对主流缓存组件进行一致性集成。通过暴露统一的接口，方便对不同组件进行随意切换。

SpringCache支持多种缓存，包括Redis。

##### 使用

1. 添加Maven依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-cache</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```

2. 在入口类添加`@EnableCaching`注解，spring cache会根据底层JAR包依赖决定使用何种缓存

3. 在application.yml中添加Redis服务器和Jedis连接池配置

4. 在方法中使用

   1. 在具体执行函数上添加`@Cacheable`注解，添加value、key参数（key以`#函数形参`的方式获取传入函数的数据），参数在存入Redis 的时候会以`value::#key`的形式作为Redis key保存到内存中

   2. 经过`@Cacheable`注解的函数在执行前会首先检查缓存，如果存在对应的key则直接取出缓存而不执行方法

   3. `@Cacheable`注解的方法返回值必须实现`Serializable`接口，不然会出现序列化错误

   4. 如果需要使用其他序列化方法，则需在入口类添加对应Bean进行配置，参考如下

      ```java
      @Bean
      public RedisCacheConfiguration redisCacheConfiguration() {
          RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig();
          redisCacheConfiguration = redisCacheConfiguration.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
          return redisCacheConfiguration;
      }
      ```

      案例使用Json序列化方式，需引入Jackson依赖

   5. `@CachePut`注解，不管是否有该缓存，总是会创建或更新Redis，多用于新建或更新函数

   6. `@CacheEvict`注解，移除目标缓存

## 持久化

### RDB

采用快照的方式进行持久化。使用Redis配置文件进行配置

常用配置项：

```
save 60 1 # 自动保存策略，每60秒有1个key放生变化则自动保存
dbfilename 文件名 # 保存的rdb文件名
stop-writes-on-bgsave-error yes # 发生错误时中断写入，建议开启
rdbcompression yes # 数据文件压缩，建议开启
rdbchecksum yes # 开启crc64错误校验，建议开启
```

### AOF

采用日志的方式。

#### AOF的追加（fsync）策略

- always - 随时写入（不推荐）
- everysec - 每秒写入（推荐）
- no - 依赖os规则吸入（不推荐）

常用配置项：

```
appendonly yes # 开启aof
appendfilename aoflog.aof # aof文件名
appendfsync everysec # aof追加策略
no-appendfsync-on-rewrite yes # 重写过程中是否继续写入日志
auto-aof-rewrite-percentage 100 # 触发重写条件：相较于上次文件增加的大小比例；默认100%
auto-aof-rewrite-min-size 64mb # 触发重写的最小文件大小
```

## 高可用和集群

### Redis主从复制

#### 特性

- 一个Master可以有多个Slave
- 一个Slave只能有一个Master
- 数据流是单向的：Master到Slave
-  主从复制底层依赖于RDB方式

#### 实现方式

- 启动多个节点

- 在从节点配置文件中添加配置：

  ```
  slaveof 127.0.0.1 6666 # 主节点地址端口
  slave-read-only yes # 是否只读
  masterauth 123456 # 主节点密码
  ```

### Sentinel

Redis Sentinel是一个分布式架构，包含若干个Sentinel节点和Redis数据节点，每个Sentinel节点会对数据节点和其他Sentinel节点做监控，当发现节点不可达时，会对节点做下线标识。

## 其他

### Redis开发规范

#### Key的设计

- 可读性，尽量按照规则：`业务名:表名:ID`
- 简洁性：建议key不要超过39个字节
- 不要包含特殊符号