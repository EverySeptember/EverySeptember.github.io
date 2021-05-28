---
title: Springboot实战笔记6
date: 2019-1-26 09:26:29
tags: Springboot
---

## 部署、打包与发布	

### 热部署（IDEA）

- 热部署前提必须是debug启动

- 关闭Thymeleaf缓存

  ```
  spring.thymeleaf.cache=false
  ```

- 在pom.xml中定义devtools，且在maven插件中添加配置类

  ```
  </dependencies>
  	<!-- 官方提供的热部署工具，用以监控class文件是否发生变化 -->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-devtools</artifactId>
      </dependency>
  </dependencies>
  
  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
              <!-- 增加fork才允许热部署，允许多个类加载器 -->
              <configuration>
                  <fork>true</fork>
              </configuration>
          </plugin>
      </plugins>
  </build>
  ```

- IDEA中的设置
  1. 需要打开`File | Settings | Build, Execution, Deployment | Compiler`中的`Build project automatically`
  2. 打开`Ctrl shift a`，找到registry，保证`compiler.automake.allow.when.app.running`打开

##### 注：

- 本方法实测只有在idea失去焦点的时候才会部署，且部署仍是重启程序，启动较慢，以后有大项目之后再观察

### Jar包与发布

首先，在pom.xml中将项目描述为`<packaging>jar</packaging>`

##### 打包

- IDEA使用maven命令的方式

  1. 在发布方式中新增maven，命令为package

     ![新增打包命令](https://ws3.sinaimg.cn/large/005BYqpggy1fzjrnk4qw7j31fu0u04ij.jpg)

  2. 运行该启动方式，生成jar包

  3. 将application.properties放到jar包同一目录下可以直接修改项目参数

- 直接使用maven插件的方式

  ![maven插件的方式](https://ws3.sinaimg.cn/large/005BYqpgly1fzjsfvjsvoj312a0kokcy.jpg)

##### 发布

- Linux下后台运行
  - 输入命令`nohup java -jar deploy-0.0.1-SNAPSHOT.jar >> out.log &`
  - nohup表示静默输出，`>>`将信息输出到指定目标文件，`&`符号不可少
  - 停止项目：直接kill

### War包与发布

首先，在pom.xml中将项目描述为`<packaging>war</packaging>`

##### 打包

1. 添加依赖，`<scope>provided</scope>`目的是仅在编译时使用该jar包，打包和运行时，不使用该jar

   ```
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-tomcat</artifactId>
       <scope>provided</scope>
   </dependency>
   ```

   - 其他scope范围
     - runtime，默认值，在编译、运行、发布时均存在
     - runtime，运行时，本地编译时不用该jar包，运行发布时加载到运行环境中

2. 将原有入口类main方法失效

   1. 入口类继承`SpringBootServletInitializer`类，该类的作用就是在tomcat启动的时候，执行内置的configure方法，将其托管给Spring Boot；同时重写configure方法，指定托管的入口类

      ```
      @SpringBootApplication
      public class DeployApplication extends SpringBootServletInitializer {
      
          @Override
          protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
              return builder.sources(DeployApplication.class);
          }
      
          public static void main(String[] args) {
              SpringApplication.run(DeployApplication.class, args);
          }
      
      }
      ```

   2. IDEA使用maven的打包配置方式与jar完全相同，可直接使用已经配置好的命令

   3. 运行该命令

##### 发布

1. 将war包放到tomcat webapps目录下
2. 启动过程中el包可能与tomcat自带el包产生冲突，可将tomcat自带该包删除
3. 启动Tomcat