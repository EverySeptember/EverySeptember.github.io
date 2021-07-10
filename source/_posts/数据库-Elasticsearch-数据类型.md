---
title: 数据库-Elasticsearch-数据类型
date: 2021-07-10 22:01:48
tags: [数据库, Elasticsearch]
---

# 数据类型

字段类型是按族分组的。同一族的类型支持相同的搜索功能，但可能有不同的空间使用或性能特征。

目前，唯一的类型族是keyword，它由keyword、constant_keyword和通配符字段类型组成。其他类型族只有一个字段类型。

## 分类

### 常见类型

- `binary`Base64编码的二进制类型
- `boolean`包含`true`和`false`
- `Keywords` `Keywords`类型族，包括`keyword`、`constant_keyword`和`wildcard`。
- `Numbers`数据类型
- `Dates`日期相关数据类型

### 对象相关

- `object`JSON对象
- `flattened`将整个JSON对象映射为一个字段
- `nested`一个保留了其子字段之间关系的JSON对象
- `join`在同一索引中定义文档的继承关系

### 结构化数据类型

- `Range`范围类型，如long_range、double_range、date_range和ip_range
- `ip`IPv4和IPv6地址
- `version`软件版本。支持Semantic Versioning优先规则。
- `murmur3`计算和存储数值的哈希值

### 聚合数据类型

- `aggregate_metric_double`预先汇总的度量值
- `histogram`以柱状图的形式预先汇总的数值

### 文本搜索类型

- `text`经过分析的，非结构化的文本
- `annotated-text`含有特殊标记的文本，用于识别命名的实体
- `completion`用于自动完成建议
- `search_as_you_type`TODO类文本类型？
- `token_count`文本中标记的计数

### 文档排行类型

- `dense_vector`记录浮点数值的密集向量
- `sparse_vector`记录浮点数值的稀疏向量
- `rank_feature`记录一个数字特征，以提高查询时的点击率
- `rank_features`记录数字特征，以便在查询时提高点击率

### 空间数据类型

- `geo_point`纬度和经度点
- `geo_shape`复杂的形状，如多边形
- `point`任意的笛卡尔点
- `shape`任意的笛卡尔几何图形

### 其他类型

- `percolator`对用查询DSL编写的查询进行索引

### 数组

在Elasticsearch中，数组并不要求有专门的字段数据类型。任何字段在默认情况下都可以包含零个或多个值，然而，数组中的所有值都必须是相同的字段类型。

### 多字段

可以对同一字段以不同的方式进行索引

