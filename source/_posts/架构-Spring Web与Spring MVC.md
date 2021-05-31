---
title: 架构-Spring Web与Spring MVC
date: 2020-06-09 16:20:12
tags: [架构,Spring,Spring Web,Spring MVC]
---

# Spring Web

# 实现

## Demo

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-web</artifactId>
       <version>4.3.27.RELEASE</version>
   </dependency>
   ```

2. 新建SpringContext类进行配置，需实现ApplicationContextAware和DisposableBean接口

   ```java
   package lu.tf.myshop.commons.context;
   
   import org.springframework.beans.BeansException;
   import org.springframework.beans.factory.DisposableBean;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.ApplicationContextAware;
   
   public final class SpringContext implements ApplicationContextAware, DisposableBean {
   
       private static ApplicationContext applicationContext;
   
       ...
   
       public void destroy() throws Exception {
           applicationContext = null;
       }
   
       public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
           SpringContext.applicationContext = applicationContext;
       }
   }
   ```

3. 在spring配置文件中注册bean，该Bean需放置在第一个以便首先进行装配

   ```xml
   <bean id="springContext" class="lu.tf.myshop.commons.context.SpringContext" />
   ```

4. web.xml添加配置

   ```xml
   <context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath:spring-context*.xml</param-value>
   </context-param>
   <listener>
       <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   </listener>
   ```

# Spring MVC

也称Spring Web MVC，属于**展示层**框架。是Spring框架的一部分。

# Demo

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-webmvc</artifactId>
       <version>4.3.17.RELEASE</version>
   </dependency>
   ```

2. 创建spring-mvc.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
               http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">
   
       <description>Spring MVC Configuration</description>
   
       <!-- 加载配置文件 -->
       <context:property-placeholder ignore-unresolvable="true" location="classpath:myshop.properties"/>
   
       <!-- 使用注解自动注册Bean，只扫描Controller -->
       <context:component-scan base-package="lu.tf.myshop" use-default-filters="false">
           <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
       </context:component-scan>
   
       <!-- 默认的注解映射支持 -->
       <mvc:annotation-driven />
   
       <!-- 定义视图文件解析 -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <property name="prefix" value="${web.view.prefix}"/>
           <property name="suffix" value="${web.view.suffix}"/>
       </bean>
   
       <!-- 静态资源映射 -->
       <mvc:resources mapping="/static/**" location="/static/" cache-period="31536000"/>
   </beans>
   ```

3. 修改web.xml的servlet由spring-mvc托管

   ```xml
   <servlet>
       <servlet-name>springServlet</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <init-param>
           <param-name>contextConfigLocation</param-name>
           <param-value>classpath*:/spring-mvc*.xml</param-value>
       </init-param>
       <load-on-startup>1</load-on-startup>
   </servlet>
   <servlet-mapping>
       <servlet-name>springServlet</servlet-name>
       <url-pattern>/</url-pattern>
   </servlet-mapping>
   ```

4. 使用注解，并使Springmvc扫描，完成映射