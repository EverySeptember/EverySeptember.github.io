---
title: SSM开发实战4-Freemarker
date: 2019-3-19 19:24:27
tags: [架构,SSM,Freemarker]
---


# 模板引擎

- JSP
- Freemarker
- Beetl
- Thymeleaf

# Freemarker

- 免费开源的模板引擎
- Freemarker脚本为FTL（**F**reemarker **T**emplate **L**anguage）
- 提供了大量内建函数简化开发

# 与Spring MVC集成

1. ## 引入依赖

   ```xml
   <dependency>
       <groupId>org.freemarker</groupId>
       <artifactId>freemarker</artifactId>
       <version>2.3.28</version>
   </dependency>
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context-support</artifactId>
       <version>5.1.5.RELEASE</version>
   </dependency>
   ```

2. ## 修改applicationContext.xml

   添加如下内容

   ```xml
   <!--配置Freemarker-->
   <bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
       <!--定义Freemarker文件放置目录-->
       <property name="templateLoaderPath" value="/WEB-INF/ftl"></property>
   </bean>
   <!--定义SpringMVC的视图解析器指向Freemarker-->
   <bean id="ViewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
       <property name="contentType" value="text/html; charset=UTF-8"></property>
       <!--设置后缀，使路径跳转自动添加.ftl后缀-->
       <property name="suffix" value=".ftl"></property>
   </bean>
   ```

3. ## 在ftl目录中新建ftl文件

# 基本语法

- ## 取值

  - ${属性名}
  - ${属性名!默认值}
  - ${属性名?string('')}-格式化输出

- ## if分支判断

  ```ftl
  <!--??表示不为空-->
  <#if u??>
  <#if (u.salary < 10000)>
      past
  <#elseif (u.salary < 15000)>
      now
  <#else>
      future
  </#if>
  </#if>
  ```

- ## 循环

  ```ftl
  <#list users as user>
      <li>${user_index}</li>
      <li>${user.name}</li>
  </#list>
  ```

- ## 内建函数

  使用方法：参数?内建函数名称

  常用内建函数：

  | 函数名                | 说明                    | 实例                   |
  | --------------------- | ----------------------- | ---------------------- |
  | lower_case/upper_case | 大小写转换              | "abc"?upper_case       |
  | cap_firset            | 首字母大写              | "lu"?cap_firset        |
  | index_of              | 查找字符索引            | "abc"?index_of("b")    |
  | length                | 返回字符串长度          | "abc"?length           |
  | round/floor/ceiling   | 四舍五入/下取整/上取整  | pi?floor               |
  | size                  | 得到集合元素总数        | users?size             |
  | fist/last             | 获得第一个/最后一个元素 | users?first            |
  | sort_by               | 按某个属性对集合排序    | list?sort_by("salary") |

  

