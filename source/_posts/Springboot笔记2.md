---
title: Springboot笔记2
date: 2019-01-10 19:19:17
tags: Springboot
---
# Springboot笔记2

## 核心特性

### Springboot三大特性

- 组件自动装配：Web MVC、Web Flux、JDBC等

- 嵌入式web容器：Tomcat、Jetty及Undertow
- 生产准备特性：指标、健康检查、外部化检查等



#### 组件自动装配

1. 激活自动装配：@EnableAutoConfiguration
2. 配置：/META-INF/spring.factories
3. 实现：XXXAutoConfiguration

#### 嵌入式web容器

- Web Servlet：Tomcat、Jetty及Undertow
- Web Reactive：Netty Web Server

#### 生产准备特性

- 指标：/actuator/metrics
- 健康检查：/actuator/health
- 外部化配置：/actuator/configprops



## Web应用

### 传统Servlet应用

#### 依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

#### Servlet组件

- Servlet
  - 实现
    - @WebServlet
    - HttpServlet
    - 注册
  - URL映射
  - 注册
- Filter
- Listener



#### Servlet注册

##### Servlet注解

- @ServletComponentScan +
  - @WebServlet
  - @WebFilter
  - @WebListener

##### Spring Bean

##### RegistrationBean

#### 异步非阻塞：异步Servlet、非阻塞Servlet

##### 异步Servlet

- javax.servlet.ServletRequest#startAsync()
- javax.servlet.AsyncContext

##### 非阻塞Servlet

- javax.servlet.ServletInputStream#setReadListener
  - javax.servlet.ReadListener
- javax.servlet.ServletOutputStream#setWriteListener
  - javax.servlet.WriteListener

### Spring Web MVC应用

#### Web MVC视图

- ViewResolver
- View

##### 模板引擎

- Thymeleaf
- Freemarker
- JSP

##### 内容协商

- ContentNegotiationConfigurer
- ContentNegotiationStrategy
- ContentNegotiatingViewResolver

##### 异常处理

- @ExceptionHandler
- HandlerExceptionResolver
  - ExceptionHandlerExceptionResolver
- BasicErrorController (Spring Boot)

#### Web MVCREST

#### Web MVC核心