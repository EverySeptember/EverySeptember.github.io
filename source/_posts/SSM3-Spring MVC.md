---
title: SSM开发实战3-Spring MVC
date: 2019-3-19 19:24:27
tags: [架构,SSM,Spring MVC]
---


## MVC

- **M**odel，模型
- **Vi**ew，视图
- **C**ontroller，控制器

## Spring MVC

- Spring体系的轻量级Web MVC框架
- 核心是Controller控制器，职责是处理请求，产生响应
- 基于IOC容器运行，所有对象被IOC管理

# 配置

## 1.引入依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.1.5.RELEASE</version>
</dependency>
```

## 2.配置web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_1.xsd"
           version="3.1">

    <display-name>springmvc</display-name>
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

    <!--Tomcat启动的时候自动初始化名为springmvc的IOC容器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!--上下文配置-->
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <load-on-startup>0</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <!--所有的请求都绑定servlet-->
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

## 3.定义applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--启用对目标的注解扫描-->
    <context:component-scan base-package="com.lu"></context:component-scan>
    <!--使用注解模式启用Spring MVC-->
    <mvc:annotation-driven></mvc:annotation-driven>
    <!--将图片、JS、CSS等静态资源排除在外-->
    <mvc:default-servlet-handler />
</beans>
```

# Spring URL Mapping

注解形式

- RequestMapping，通用绑定
- PostMapping，Post请求绑定
- GetMapping，Get请求绑定

# Controller接收请求数据

- 直接使用Controller方法参数接收

- 使用Java Bean接收数据

  - 日期格式的字段需要使用`@DateTimeFormat(pattern = "yyyy-MM-dd")`注解来描述日期格式

- 使用路径变量接收数据

  - 在Mapping接收地址中使用大括号接收路径变量
  - 在方法参数使用`@PathVariable("路径变量名")`注解标注并传值给注解的形参

  ```java
  @GetMapping("/hi/{name}")
  public String hi(@PathVariable("name")String name) {
      return "hi " + name;
  }
  ```

# 中文乱码的配置解决

- Get请求乱码：在Tomcat server.xml的Connector标签中添加属性URIEncoding

      <Connector port="8080" protocol="HTTP/1.1"
                 connectionTimeout="20000"
                 redirectPort="8443" 
      		   URIEcoding="UTF-8"/>

- Post请求乱码：在web.xml中配置添加过滤器CharacterEncodingFilter

  ```xml
  <!--过滤器-->
  <filter>
      <filter-name>characterFilter</filter-name>
      <!--springframework自带的编码过滤器-->
      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
      <init-param>
          <!--设置目标编码-->
          <param-name>encoding</param-name>
          <param-value>UTF-8</param-value>
      </init-param>
  </filter>
  <!--设置过滤器映射-->
  <filter-mapping>
      <filter-name>characterFilter</filter-name>
      <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

- Response响应乱码：配置applicationContext.xml中spring mvc

  ```xml
  <!--使用注解模式启用Spring MVC-->
  <mvc:annotation-driven>
      <mvc:message-converters>
          <bean class="org.springframework.http.converter.StringHttpMessageConverter">
              <property name="supportedMediaTypes">
                  <list>
                      <value>text/html;charset=utf-8</value>
                  </list>
              </property>
          </bean>
      </mvc:message-converters>
  </mvc:annotation-driven>
  ```

# 响应的处理方式

- @ResponseBody：产生相应文本

  - SpringMVC支持对象序列化，需要引入相应的JAR包实现自动序列化
  - 以Jackson为例

  ```xml
  <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.9.8</version>
  </dependency>
  <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.9.8</version>
  </dependency>
  <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.8</version>
  </dependency>
  ```

- ModelAndView对象：套用模板引擎渲染输出

  ```java
  public ModelAndView findByUser(String username) {
      User user = userService.findByUserName(username);
      ModelAndView modelAndView = new ModelAndView("/info.jsp");
      modelAndView.addObject("u", user);
      return modelAndView;
  }
  ```

