---
title: Springboot实战笔记2
date: 2019-1-16 19:20:59
tags: Springboot
---

## Thymeleaf模板引擎入门

### Thymeleaf的特点

- Thymeleaf优点
  - 主流唯一的前后端通用引擎，静态html嵌入标签属性，浏览器可以直接打开模板文件，便于前后端联调
  - springboot官方推荐方案
- Thymeleaf缺点
  - 模板必须返回xml规范
  - 慢


### 使用Thymeleaf

1. 引入Thymeleaf的命名空间

   ```
   <html xmlns:th="http://www.thymeleaf.org">
   ```

2. 使用th标签

3. 调用格式

   ```
   th:标签="#{属性}"
   #{}表示读取常量
   ${}表示读取变量
   ```

#### Thymeleaf迭代

```
th:each="object,stat:${list}"、
其中：
	object是每次迭代产生的对象
	list是被迭代的数据集
	stat表示当前的迭代状态，会有许多属性
```

#### 格式化 

使用#strings，#dates，#numbers等方法

[Thymeleaf官方文档]: https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-b-expression-utility-objects

#### 分支与判断

- th属性中使用三目运算符
  - `th:text="${emp.comm} != null ? ${emp.comm} : 'N/A'"`
  - 表达方式与java相同
- th:if / th:unless判断标签是否输出
  - `th:if="${size > 0}"`
  - `th:unless="${size > 0}"`
  - if在判断为真的时候成立，unless在判断为假的时候成立
- 多分支判断
  - `th:swith="${}" th:case=""`
  - `th:case="*"`表示除已列出以外情况的所有其他情况

#### 创建模板与引用模板

1. 新建一个html文件
2. 在要创建为模板的标签中添加`th:fragment="模板名称" `
3. 在引用文件中使用`<div th:insert="模板文件名 :: 模板名称"></div>`调用模板

#### 获取请求参数

- `${param.xxx}`用于获取请求参数
  - 相当于`request.getParameter('xxx')`