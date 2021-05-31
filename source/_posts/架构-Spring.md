---
title: 架构-Spring
date: 2020-06-09 16:20:12
tags: [架构,Spring]
---


# 特点

## 非侵入式

Spring框架的API不会在业务逻辑上出现，即业务逻辑代码与环境无关。

## 容器

Spring作为一个容器，可以管理对象的生命周期、对象与对象之间的依赖关系。可以通过配置文件，来定义以及设置对象与其他对象的依赖关系。

## IoC

控制反转，即创建被调用者的实例由Spring完成并注入。

当应用了IoC，一个对象依赖的其他对象会通过被动的方式传递进来，而不是通过调用者自己创建或者查找依赖。

### 实现方式

- **依赖注入**：Dependency Injection，DI，程序代码不做定位查询工作，而是由容器完成这些工作。
- 依赖查找（不常用）：Dependency Lookup，DL，容器提供回调接口和上线文环境给组件，程序代码则需要提供具体的查找方式。

## AOP

面向切面编程，是一种编程思想，是面向对象编程的补充。Spring也提供了面向切面编程的支持，允许分离应用的业务逻辑与系统级服务来进行开发。即只关注业务逻辑的实现，而不必关注系统服务的实现。

# 实现

## Demo

### ApplicationContext方式

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context</artifactId>
       <version>4.3.27.RELEASE</version>
   </dependency>
   ```

2. 配置文件：resources/spring-context.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
   </beans>
   ```

3. 业务代码

4. 在spring配置文件中注册bean

   ```xml
   <bean id="userService" class="lu.tf.hello.spring.service.impl.HelloServiceImpl" />
   ```

5. 获取Bean

    ```java
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-context.xml");
         HelloService userService = (HelloService) applicationContext.getBean("userService");
    ```
    
### 注解方式

1. 添加依赖

2. 在spring-context中开启注解模式与包扫描

    ```xml
    <context:annotation-config />
    <context:component-scan base-package="包名" />
    ```

3. 业务代码

4. 添加注解

5. 使用

    ```java
    @Resource
    private HelloService helloService;
    ```

# Bean

## 作用域

- singleton：单例（默认），全局仅一个bean
- prototype：原型，每次访问都会创建一个新的实例
- request：对于每次HTTP请求，都会产生一个新的实例
- session，每个不同的HTTP Session，都会产生一个新的实例
- global session，每个全局的HTTP Session对应一个实例（在portlet集群时有效）

