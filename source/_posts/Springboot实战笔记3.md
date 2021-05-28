---
title: Springboot实战笔记3
date: 2019-1-17 20:03:08
tags: Springboot
---

# Springboot实战笔记3

## Web开发

- Spring Boot帮我们简化了构架的依赖与配置过程，但在Web开发层面上，让然采用Spring MVC开发的方式
- Spring MVC中最重要的环节就是开发Controller控制器

##### 上下文数据

以下三种方式传值方式

- ModelAndView（推荐）
- Model
- WebRequest或者原生的HttpServletRequest对象（不推荐）

##### 返回方式

使用`@ResponseBody`可以将返回值转为json，SpringBoot默认使用的json转化工具是jackson

##### 自定义404和500错误页面配置

在templates文件夹下新建error文件夹，在error中分别创建404.html和500.html，则可自动配置

##### controller注解

在controller类上使用`@RestController`代替`@Controller`可以使本类中的所有方法默认返回json而不是页面，相当于`@ResponseBody + @Controller`。

### 文件上传

##### 前台配置

文件上传前台需要满足的三个条件

- post提交
- 具备file组件
- 表单上传方式设置为`enctype="multipart/form-data"`

##### 后台接收

- 使用`@PostMapping`注解，接收post数据
- 使用`@RequestParam`注解，与前端name对应一致
- `MultipartFile`是一个文件接口，保存了文件上传的数据

```
@PostMapping("/create")
public ModelAndView create(@RequestParam("photo") MultipartFile param) {
    ...
}
```

`FileCopyUtils`是Spring Boot提供的一个专门的文件复制类，可使用进行文件复制，将文件从temp文件夹放到目标文件夹。

##### 上传配置

SpringBoot默认的文件上传单个文件最大为1M，单次请求最大数据量为100M；可在properties文件中进行配置

```
# 单个文件最大数据
spring.servlet.multipart.max-file-size=5mb
# 单词请求最大数据
spring.servlet.multipart.max-request-size=50mb
# 设置上传默认文件夹
spring.servlet.multipart.location=C:/Users/Lu/Desktop/uploaded/temp
```

### Filter

1. 自定义Filter类，实现`javax.servlet.Filter`接口
2. 在入口类添加方法，其返回类型为`FilterRegistrationBean`，注解使用`@Bean`

```
@Bean // @Bean注解的作用是在SpringBoot启动时将方法的返回值放到IoC中
public FilterRegistrationBean filterRegiste() {
   FilterRegistrationBean registrationBean = new FilterRegistrationBean();
   // 创建并注册，AccessRecorderFilter是自定义的Filter类
   registrationBean.setFilter(new AccessRecorderFilter());
   registrationBean.addUrlPatterns("/*");
   registrationBean.setName("AccessRecorder");
   // 设置排序，如果有多个过滤器，order越小的越优先执行
   registrationBean.setOrder(1);
   return registrationBean;
}
```

### Web容器

- Tomcat：默认的
- Jetty：性能优秀的内嵌Web容器，适用于长连接
- Undertow：非阻塞Web容器，性能优异，适用于高并发

#### 替换web容器

1. 在pom.xml中移除spring-boot-starter-web对Tomcat的依赖
2. 添加对其他容器的依赖

```
<dependencies>
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-thymeleaf</artifactId>
   </dependency>

   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jetty</artifactId>
   </dependency>

   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <exclusions>
         <!-- 移除原来对tomcat的依赖 -->
         <exclusion>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
         </exclusion>
      </exclusions>
   </dependency>

   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
   </dependency>
</dependencies>
```

