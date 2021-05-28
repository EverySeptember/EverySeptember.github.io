---
title: Springboot笔记
date: 2019-01-05 13:39:44
tags: Springboot笔记
---

### Spring Data

使用Spring Data（一种借口）替代dao层，可有效提高代码复用。

### DDD模型（领域驱动设计）

它像是更小粒度的迭代设计，它的最小单元是`领域模型(Domain Model)`，所谓领域模型就是能够精确反映领域中某一知识元素的载体，这种知识的获取需要通过与`领域专家(Domain Expert)`进行频繁的沟通才能将专业知识转化为领域模型。领域模型无关技术，具有高度的业务抽象性，它能够精确的描述领域中的知识体系；同时它也是独立的，我们还需要学会如何让它具有表达性，让模型彼此之间建立关系，形成完整的领域架构。通常我们可以用象形图或一种`通用的语言(Ubiquitous Language)`去描述它们之间的关系。在此之上，我们就可以进行`领域中的代码设计(Domain Code Design)`。

https://www.jianshu.com/p/b6ec06d6b594

### @RestController

可以拆分成两个注解@ResponseBody及@Controller，分别表示可返回对象以及这是一个控制器。

### Repository对Controller采用构造器的方式注入

构造器的方式使Repository不可修改，且可以提早进行初始化。

### 转发与接收请求

@PostMapping转发请求

@RequestParam可以自动匹配参数



## Web Flux

### 概念

传统NIO：同步非阻塞；

Reactor：异步非阻塞的Reactive实现；

Flux：0到N的对象；

Mono：0到1的对象；

### 路由函数

采用路由函数可替代Mapping注解。

### λ表达式个人理解

`(parameters) ->{ statements; }`

λ表达式是一个匿名函数，返回的是抽象方法的具体实现；parameters是该方法的参数列表，statements即该方法的方法体。



## 多模块

在主工程pom.xml中调整packaging为<packaging>pom</packaging>

拆分为web、persistence、model三个模块，并根据pom.xml处理相关依赖

```
模型层：model
持久层：persistence
表示层：web
web层依赖于persistence，persistence依赖于model
```



## 项目打包

1. 指定Main-Class
   - 将原来在project的pom.xml的plugins配置放到主模块的pom.xml中去
2. 打包
   - JAR、WAR
   - 通过修改主module中pom.xml中packaging的形式修改打包方式
   - 使用IDEA打包
     - 坑：https://blog.csdn.net/u011624972/article/details/58591825