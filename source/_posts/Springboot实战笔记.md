---
title: Springboot实战笔记
date: 2019-1-14 18:49:49
tags: Springboot
---

# Spring Boot常用配置

## 目录结构

- java：源码
- resource：资源位置
  - static：静态资源（js、css、图片等）
  - template：动态页面（模板文件位置）
  - application.properties：核心配置文件

## application.properties

### 常用配置项

| web常用配置项                | 默认值 | 说明              |
| :--------------------------- | :----- | :---------------- |
| debug                        | false  | 开启/调试模式关闭 |
| server.port                  | 8080   | 服务器端口        |
| server.servlet.context-path  | /      | 应用上下文        |
| spring.http.encoding.charset | utf-8  | 默认字符集编码    |
| spring.thymeleaf.cache       | true   | 开启/关闭页面缓存 |
| spring.mvc.date-format       |        | 日期输入格式      |
| spring.jackson.date-format   |        | json输出的日期    |
| spring.jackson.time-zone     |        | 设置GMT时区       |

### PS

- UTF-8只包含了20000+个中文字符，对于生僻字显示不了
- 开发时关闭Thymeleaf缓存，同时配合自动构建，可以实现热部署，提高开发效率

## 日志

Springboot默认使用logback

### 常用配置项

| 日志常用配置项     | 默认值             | 说明                 |
| ------------------ | ------------------ | -------------------- |
| logging.file       |                    | 日志输出的文件       |
| logging.level.ROOT | info               | 设置日志的输出级别   |
| logging.level.*    | info               | 定义指定包的输出级别 |
| logging.config     | logback-spring.xml | 日志的配置文件       |

### PS

- 日志级别 debug->info->warn->error，默认级别为info，设置debug为true，会出现debug以上的级别
- logging.level.root代表全局设置，只会显示root配置级别及以上的日志；logging.level.*可以指定包的显示级别
- 指定logging.config参数后application.properties中的日志配置将会失效，转而使用配置文件中的设置



## Spring Boot配置文件

SpringBoot支持两种格式的配置文件

- 属性文件：application.properties
- yaml：application.yml

当同时存在两种格式文件，以properties为准

### yaml

yaml是一种简洁的非标记语言。yaml以数据为中心，使用空白、缩进、分行组织数据，从而更加简洁易读。

#### yaml语法格式

- 标准格式：key:（空格）value
- 使用空格代表层级关系，以`:`结束

## 环境配置文件

- SpringBoot可针对不同的环境提供不同的Profile文件
- Profile文件的默认命名格式为application-{env}.yml
- 使用spring.profiles.active选项来指定不同的profile
- 不同环境日志系统的配置需要用logback-spring.xml修改

## Spring Boot自定义配置

Spring Boot内置的配置项远远不能支撑我们的程序运行，在项目设计的时候，往往因为扩展性的需要，项目需要预留很多自定义设置项，Spring Boot允许我们配置自定义选项。

### 配置方式

- @Value单个属性注入
  - 在类属性添加`@Value(${name})`注解，便可将不同的配置注入到当前属性中
- @ConfigurationProperties类型安全加载
  - 在实体类添加`@Component`，表明这是一个组件类，这样Spring Boot启动时会加载该类
  - 在实体类添加`@ConfigurationProperties(prefix="app")`，可以添加所有配置中前缀为app的配置，并自动与属性匹配（需实体类属性与配置文件属性命名方式相对应）
  - 在调用实体类位置添加`@Autowired`或`@Resource`注解，即可实现动态注入