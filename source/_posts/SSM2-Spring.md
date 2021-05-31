---
title: SSM开发实战2-Spring
date: 2019-3-19 19:24:27
tags: [架构,SSM,Spring]
---


# IOC控制反转

- 全称Inverse Of Control，设计一种设计理念。
- 核心是由代理人创建与管理对象，消费者通过代理人获取对象。
- 目的是降低程序之间的直接耦合。

# DI依赖注入

- IOC是设计标准
- DI(Dependency Injection)是具体实现
- 在Java中DI是利用反射实现的

# Spring Framework

- Spring框架是IOC理念的具体实现
- Spring核心技术是反射
- 设计模式主要使用工厂与代理模式实现

# 1.Spring创建（xml方式）

## 1.引入JAR包

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.5.RELEASE</version>
</dependency>
```

## 2.创建配置文件

在src/main/resources下创建applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
</beans>
```

## 3.创建入口类

在入口类中添加main方法，在项目启动时，便会根据加载内容对IOC容器进行初始化

```java
public static void main(String[] args) {
    // 加载核心配置文件
    ApplicationContext atx = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
}
```

## 4.创建类

在类中创建要加载的类作为私有属性，并且创建对应的set方法；spring IOC容器初始化时会使用该set方法注入要加载的类。

## 5.在核心配置文件中配置要加载的类

```xml
<bean id="empDao" class="com.lu.spring.dao.EmpDao"></bean>
<bean id="empService" class="com.lu.spring.service.EmpService">
    <!--配置类中私有属性与bean的映射关系-->
    <!--name是类中的属性，ref是指向的bean-->
    <property name="dao" ref="empDao"></property>
</bean>
```

# 2.Spring创建（注解方式）

## Spring IOC注解

- @Responsitory-持久层类
- @Service-业务逻辑层
- @Controller-控制器类
- @Component-组件类
- @Resource-智能加载
- @Autowired-按类型加载

## 1.引入依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.5.RELEASE</version>
</dependency>
```

## 2.创建配置文件

在src/main/resources下创建applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!--
        扫描目标包下的所有类，并将拥有以下注释的类进行加载
        @Reponsitory - 持久层类
        @Service - 业务逻辑
        @Controller - 控制器
        @Component - 组件
    -->
    <context:component-scan base-package="com.lu.spring"></context:component-scan>
</beans>
```

## 3.创建类，并将类使用对应的注解

使用注解创建的bean，id默认为首字母小写的类名；在注解后也可以对bean id进行命名，如`@Responsitory("eDao")`

## 4.注入对象

### @Resource

在类中创建对象，无须创建get、set方法，使用@Resource注解注入。

1. 默认Resource注解使用属性名作为bean id进行注入
2. 指定name属性，可以按照name进行加载，如`@Resource(name = "eDao")`
3. （不推荐）未指明name，同时也不存在属性名相同的bean时，则自动按照类型进行匹配

### @Autowired

只能按照类型进行注入，已不常用

# AOP面向切面编程

在不修改源代码的情况下为程序进行扩展的计数

## 1.引入依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>5.1.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.1</version>
</dependency>
```

## 2.修改配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="com.lu.spring"></context:component-scan>
    <!--启用AOP注解功能-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

## 3.创建切面类

切面类需要@Component和@Aspect注解

最强力的注解：@Around注解，使用在方法上，环绕通知

```java
package com.lu.spring.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class EnhanceAspect {

    /**
     * 环绕通知
     * Around注解：应用在当前工程的所有Service类的所有方法中
     * @param pjp
     * @return
     * @throws Throwable
     */
    @Around("execution(* com.lu..*Service.*(..))")
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        // 前置通知
        System.out.println(pjp.getTarget().getClass().getSimpleName() + "." + pjp.getSignature().getName());
        // 执行目标方法
        Object ret = pjp.proceed();
        // 后置通知
        System.out.println(pjp.getTarget().getClass().getSimpleName() + "." + pjp.getSignature().getName() + "执行完成");
        return ret;
    }
}
```