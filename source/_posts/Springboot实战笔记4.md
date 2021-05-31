---
title: Springboot实战笔记4
date: 2019-1-24 19:41:09
tags: Springboot
---


# Spring Data与JPA

Spring Data项目的目的是简化构建基于Spring框架应用的数据访问技术，包括关系型数据库、非关系型数据库、Map-Reduce框架、云数据服务等；

JPA是Java Persistence API的简称，中文名是Java持久层API，是JDK 5.0注解或XML描述对象-关系表的映射关系，并将运行期的实体对象持久化到数据库中。

SpringData JPA是SpringData下的一个重要模块，负责关系型数据库的映射；实现了Java JPA标准。

SpringData默认的JPA实现是Hibernate。

#### Hibernate优缺点

- 优点
  - 面向对象
  - 更好的移植性
  - 开发效率高
- 缺点
  - 运行效率慢
  - 结构臃肿
  - JPQL/HQL存在硬伤
- 使用建议
  - 用户量不大，或许要敏捷开发的企业级应用
  - 互联网项目慎用



## 基本操作CRUD

1. #### 在application.properties配置数据库连接配置

   ```
   spring.datasource.driver-class-name=com.mysql.jdbc.Driver
   spring.datasource.url=jdbc:mysql://127.0.0.1:3306/scott?useUnicode=true&characterEncoding=utf-8
   spring.datasource.username=root
   spring.datasource.password=root
   # show-sql可以在运行时展示sql语句
   spring.jpa.show-sql=true
   ```

2. #### 创建实体类

   - 实体类使用`@Entity`和`@Table(name="")`注解，`@Entity`是实体类标识；SpringBoot启动时会加载这个类，`@Table`注解使表与表明一一对应
   - 字段使用`@Column(name="")`注解，如果属性名与字段名相同则可以省略
   - 在属性上添加`@Id`注解表示主键，使用`@GeneratedValue`表示序列增长模式
     - `@GeneratedValue(strategy = GenerationType.IDENTITY)`使用数据库自增
     - `@GeneratedValue(strategy = GenerationType.SEQUENCE)`使用序列增长

3. 创建Repository接口（类似于dao层接口）

   - Repository接口继承org.springframework.data.jpa.repository.JpaRepository，则自动提供增删改查方法
   - JpaRepository传入泛型为需要关联的实体以及该实体主键的数据类型

   ```
   import com.lu.springdatajpa.entity.Dept;
   import org.springframework.data.jpa.repository.JpaRepository;
   
   public interface DeptRepository extends JpaRepository<Dept, Integer> {
   }
   ```

4. 创建Controller（以查询为例）

   - 在controller中注入Repository接口

     ```
     @Resource
     private DeptRepository deptRepository = null;
     ```

   - Repository接口查询返回值为`Optional<T>`，是实体类的包装类

     ```
     Optional<Dept> op = deptRepository.findById(id);
     Dept dept = null;
     // op.isPresent()判断是否对象存在
     if (op.isPresent()) {
     	// op.get()方法可以直接获得实体类对象
         dept = op.get();
     }
     ```

   - Repository的创建和保存方法都是`save(T)`，根据是否拥有主键判断，如果有则新建，否则更新

   - 删除方法为`delete(T)`，同时返回主键值

   - 方法中可以使用`/{参数名}`注解来直接传值，在参数栏内使用`@PathVariable`直接引用，而前台不用输入参数

     ```
     @GetMapping("/{id}")
     public T findById(@PathVariable("id") Integer id){
     	...
     }
     ```



## 自定义查询

SpringData JPA支持自定义查询方法，在Repository接口中按照规则命名抽象方法，不用提供实现。

[spring data jpa方法命名规则](https://blog.csdn.net/sbin456/article/details/53304148)

一般情况下不推荐使用，因为复杂查询使得命名方法过于复杂。

### JPQL

一种类SQL，在Repository接口中定义抽象方法，无命名规则，且使用`@Query`进行注解。

使用`:命名参数`进行传值。

```
@Query("select d from Dept d where d.dname = :dname")
public List<Dept> findByDname(String dname);
```

#### 注意事项

- 大多数情况下将*替换为别名或直接使用表名
- 表名改为实体类名
- 字段名改为属性名



## 关系映射Mapping

在实体表中，添加主子表对象（在主表中添加子表时使用对象替换外键属性），并使用`@ManyToOne`（或其他关联关系描述）、`@JoinColumn(value = "主表关联字段名")`对该对象进行描述。

在JPQL中使用关联表的字段用以替换外键属性，如：

```
// dept是Emp的一个对象属性，使用dept中的deptno用于替换Emp中的外键
@Query("select e from Emp e where e.dept.deptno = :deptno")
public List<Emp> findAllByDeptno (Integer deptno);
```



## 连接池与Druid

### SpringBoot对连接池的支持

- 目前SpringBoot默认支持的连接池有dbcp、dbcp2、tomcat、hikari
- 数据库连接可以使用DataSource池进行自动配置
- 优先使用的连接池技术顺序
  1. Tomcat
  2. HikariCP
  3. Commons DBCP
  4. Commons DBCP2

###  Druid

- Druid是阿里巴巴提供的数据库连接池，能够提供强大的监控和扩展功能

- [Druid GitHub地址](https://github.com/alibaba/druid)、[Spring Boot Starter Druid](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)

- [DruidDataSource配置属性列表](https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8)

- 依赖

  ```
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
      <version>1.1.10</version>
  </dependency>
  ```

#### 程序中使用Druid提供的图形化分析工具

1. 在程序入口类中添加如下代码

   ```
   @Bean
   @ConfigurationProperties(prefix = "spring.datasource")
   public DataSource druid(){
       return new DruidDataSource();
   }
   
   @Bean
   public ServletRegistrationBean statViewServlet() {
       // 创建StatViewServlet，绑定到/druid/*路径下
       ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
       Map<String, String> param = new HashMap<String, String>();
       // 设置后台访问所需的账号密码
       param.put("loginUsername", "admin");
       param.put("loginPassword", "123456");
       // 允许访问后台的IP地址，“”代表所有均允许
       param.put("allow", "");
       // 禁止访问的IP地址
       param.put("deny", "10.10.10.10");
       bean.setInitParameters(param);
       return bean;
   }
   
   @Bean
   public FilterRegistrationBean webStatFilter() {
       FilterRegistrationBean bean = new FilterRegistrationBean();
       bean.setFilter(new WebStatFilter());// 设置过滤器，WebStatFilter是阿里巴巴提供的默认过滤器
       bean.addUrlPatterns("/*");
       Map<String, String> param = new HashMap<String, String>();
       // 排除静态资源
       param.put("exclusion", "*.js,*.css,*.jpg,/druid/*");
       bean.setInitParameters(param);
       return bean;
   }
   ```

2. 启动项目，访问`地址/druid`



## 声明式事务

- 默认情况下，数据库的事务作用范围是在JpaRepository的CURD方法上。

  在类或者方法上使用`org.springframework.transaction.annotation.Transactional`注解，可以使事务提高到该类或者方法。所有RuntimeException及其子类的异常均会被回滚。

  这种使用注解的注解方式，成为**声明式事务**。

  ```
  @Transactional
  public void imp() {
  	...
  }
  ```

- 可以使用`@Transactional(rollbackFor = Exception.class)`来指定触发回滚的异常等级

- 可以使用`@Transactional(propagation = Propagation.NOT_SUPPORTED, readOnly = true)`关闭事务

- 一般情况下，事务处理都要在service层