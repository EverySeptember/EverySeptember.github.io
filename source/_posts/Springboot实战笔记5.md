---
title: Springboot实战笔记5
date: 2019-01-24 19:43:48
tags: Springboot零散笔记
---

## MyBatis

MyBatis是一款优秀的持久层框架，它支持定制化SQL、存储过程以及高级映射。MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。MyBatis可以使用简单的XML或者注解来映射原生信息，将接口的POJOs（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录。

### 使用步骤

1. 引入依赖

   ```
   <dependency>
       <groupId>org.mybatis.spring.boot</groupId>
       <artifactId>mybatis-spring-boot-starter</artifactId>
       <version>2.0.0</version>
   </dependency>
   ```

2. 配置application.properties

   ```
   mybatis.config-location=classpath:/mybatis/mybatis-config.xml
   mybatis.mapper-locations=classpath:/mybatis/mapper/*.xml
   ```

3. 配置mybatis-config.xml

   ```
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   
   <configuration>
   	...
   </configuration>
   ```

4. 开发应用

   1. 创建mapper包，在包中添加相应实体的mapper接口类

   2. 在`/mybatis/mapper/`路径配置相应于接口的xml配置文件

      ```
      <?xml version="1.0" encoding="UTF-8"?>
      <!DOCTYPE mapper
              PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
              "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
      <!--映射文件配置，namespace指向接口-->
      <mapper namespace="com.lu.springbootmybatis.mapper.EmpMapper">
          ...
      </mapper>
      ```

   3. 在service中注入mapper接口类，直接使用方法

   4. 在SpringBoot入口类中添加`@MapperScan("com.lu.springbootmybatis.mapper")`注解，指向mapper接口类的路径，使其在项目启动时被加载，从而项目启动时MyBatis会实现接口类中的方法

### 查询

- xml文件配置

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <!--映射文件配置，namespace指向接口-->
  <mapper namespace="com.lu.springbootmybatis.mapper.EmpMapper">
      <!--
          select 表示查询，
          id要与接口中的方法名对应上，
          parameterType是该方法参数类型，
          resultType指定返回值类型，以完成自动注入
          #{value}即为参数值
      -->
      <select id="findById" parameterType="Integer" resultType="com.lu.springbootmybatis.entity.Emp">
          select * from emp where empno = #{value}
      </select>
  </mapper>
  ```

- 在xml文件中定义`resultType="java.util.Map"`可以有效扩展多表返回值

- `parameterType="java.util.Map"`可以传多个参数到SQL语句中，直接使用`#{key}`便可以自动对应，key表示参数map的key

- MyBatis默认不输出SQL到日期，对mapper包调整日志输出等级便可打印SQL

  ```
  logging.level.com.lu.springbootmybatis.mapper=debug
  ```

##### 动态查询

使用`<if test=""></if>`标签

### 创建数据

```
<insert id="insert" parameterType="com.lu.springbootmybatis.entity.Emp">
    INSERT INTO `scott`.`emp`(`ename`, `job`, `mgr`, `hiredate`, `sal`, `comm`, `deptno`)
    VALUES (#{ename}, #{job}, #{mgr}, #{hiredate}, #{sal}, #{comm}, #{deptno})
    <selectKey keyProperty="empno" keyColumn="empno" resultType="Integer" order="AFTER">
        select LAST_INSERT_ID()
    </selectKey>
</insert>
```

- 使用insert标签，标签内部直接写SQL

- Mapper方法中参数必须是对应的实体类，SQL占位符名称用实体类属性名
- 在insert标签中添加`<selectKey></selectKey>`获取最新插入的ID，order="AFTER"表示插入数据之后查询ID，select LAST_INSERT_ID()是mysql的查询方法

### 删改

分别使用update和delete标签即可

```
<update id="update" parameterType="com.lu.springbootmybatis.entity.Emp">
    update `scott`.`emp` set `ename` = #{ename}, `job` = #{job}, `mgr` = #{mgr},
    `hiredate` = #{hiredate}, `sal` = #{sal}, `comm` = #{comm}, `deptno` = #{deptno}
    where `empno` = #{empno}
</update>

<delete id="delete" parameterType="Integer">
    delete from emp where empno = #{empno}
</delete>
```