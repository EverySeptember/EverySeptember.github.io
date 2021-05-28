---
title: SSM开发实战5-SSM
date: 2019-3-19 19:24:27
tags: [架构,SSM]
---

# SSM开发实战5-SSM

## 步骤

1. ### 引入依赖

   ```xml
   <!--mybatis与spring整合插件-->
   <dependency>
       <groupId>org.mybatis</groupId>
       <artifactId>mybatis-spring</artifactId>
       <version>1.3.2</version>
   </dependency>
   ```

2. ### 修改spring核心配置文件applicationContext.xml

   ```xml
   <!--mybatis整合配置-->
   <!--1.配置数据源-->
   <bean id="datasource" class="com.alibaba.druid.pool.DruidDataSource">
       <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
       <property name="url" value="jdbc:mysql://39.104.126.40:3306/next-shop?useUnicode=true&amp;characterEncoding=utf-8"></property>
       <property name="username" value="root"></property>
       <property name="password" value="soulOperation@0!8"></property>
   </bean>
   <!--2.配置SessionFactory-->
   <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
       <!--创建数据库连接-->
       <property name="dataSource" ref="datasource"></property>
       <!--mybatis配置文件地址-->
       <property name="configLocation" value="classpath:mybatis-config.xml"></property>
       <!--定义扫描mapper配置文件地址-->
       <property name="mapperLocations" value="classpath:mappers/*.xml"></property>
   </bean>
   <!--3.mybatis扫描mapper接口配置-->
   <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
       <property name="basePackage" value="com.lu.ssm"></property>
   </bean>
   ```

3. ### 创建mybatis核心配置文件

   mybatis-config中关于数据库连接的部分已经交由applicationContext.xml处理

4. ### 创建mapper包下的mapper接口

   ```java
   public interface AreaMapper {
       public List<Area> findAll();
   }
   ```

5. ### 创建对应的mappers下的mapper.xml配置文件

   ```xml
   <!--namespace一定要指向mapper接口-->
   <mapper namespace="com.lu.ssm.mapper.AreaMapper">
       <!--ID要与mapper接口中的方法名一致-->
       <select id="findAll" resultType="com.lu.ssm.entity.Area">
           SELECT * FROM bas_area
       </select>
   </mapper>
   ```

# MyBatis Plus

## 特性

- 无侵入-不修改mybatis代码
- 损耗小
- 强大的CURD
- 支持多种数据库
- 内置分页功能
- 内置主键生成策略
- *默认在ssm环境执行

## 步骤

1. ### 引入依赖

   ```xml
   <dependency>
       <groupId>com.baomidou</groupId>
       <artifactId>mybatis-plus</artifactId>
       <version>3.1.0</version>
   </dependency>
   ```

2. ### 修改applicationContext.xml

   修改关于sqlSessionFactory的创建类，无须做其他任何更改

   ```xml
   <bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean">
   	...
   </bean>
   ```

3. ### 修改实体类

   - 使用@TableName()注解类，使该类与表关联
   - 使用@TableId(type = )注解主键，标注主键与主键的生成方式

4. ### 修改Mapper类

   Mapper类需继承`BaseMapper<T>`，T表示对应实体类

5. ### 修改*mapper.xml

   mapper中仅需mapper标签，mybatis plus会实现绝大部分的方法

## 条件查询

[条件构造器](https://mybatis.plus/guide/wrapper.html)

## 分页查询

1. ### 在mybatis-config.xml中添加插件

   ```xml
   <plugins>
       <plugin interceptor="com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor"></plugin>
   </plugins>
   ```

2. 使用IPage对象

   ```java
   public IPage<Area> selectByPage(Integer pageNo, Integer lines) {
       IPage<Area> page = new Page<Area>(pageNo, lines);
       QueryWrapper<Area> queryWrapper = new QueryWrapper<Area>();
       page = areaMapper.selectPage(page, queryWrapper);
   
       return page;
   }
   ```

