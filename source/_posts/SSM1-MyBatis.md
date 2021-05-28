---
title: SSM开发实战1-MyBatis
date: 2019-3-14 19:24:27
tags: [架构,SSM,MyBatis]
---

# SSM-MyBatis

## MyBatis

#### 特点

- 优秀的持久层框架
- 使用XML将SQL与程序解耦，便于维护
- 学习简单，执行高效

#### 使用步骤

![步骤](https://ws3.sinaimg.cn/large/005BYqpgly1g18elgio1pj30x80ky1jz.jpg)

1. 引入MyBatis依赖
2. 创建MyBatis配置文件
3. 创建实体（Entity）类
4. 创建Mapper映射文件
5. 初始化SessionFactory
6. 利用SqlSession对象操作数据

### 1.引入MyBatis依赖

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.0</version>
</dependency>
```

### 2.创建MyBatis配置文件

在main/resources目录下创建mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!-- 配置 -->
    <settings>
        <!-- 开启下划线转驼峰功能 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <!--配置环境-->
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"></transactionManager>
            <!--配置数据库连接-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://39.104.126.40:3306/next-shop?useUnicode=true&amp;characterEncoding=UTF-8" />
                <property name="username" value="root" />
                <property name="password" value="soulOperation@0!8" />
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mappers/AreaMapper.xml" />
    </mappers>
</configuration>
```

### 3.创建实体（Entity）类

创建Entity包，在包下创建与表对应的实体类

### 4.创建Mapper映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="AreaMapper">
	...
</mapper>
```

### 5.初始化SessionFactory

```java
Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
```

读取MyBatis核心配置文件，使用SessionFactory加载配置

### 6.利用SqlSession对象操作数据

```java
public void testFindAll() throws IOException {
    Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);

    // 打开session
    SqlSession sqlSession = sqlSessionFactory.openSession();
    // 读取Mapper映射文件中的配置
    List<Area> list = sqlSession.selectList("AreaMapper.findAll");
    for (int i = 0; i < list.size(); i++) {
        Area area = list.get(i);
        System.out.println(area.getAreaName() + "-" + area.getAddress());
    }

    sqlSession.close();
}
```
