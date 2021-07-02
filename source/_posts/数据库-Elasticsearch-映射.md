---
title: 数据库-Elasticsearch-映射
date: 2021-06-29 09:40:00
tags: [数据库, Elasticsearch]
---

# 映射

## 动态映射

自动检测和添加新字段的做法被称为动态映射。通过将`dynamic`设置为`true`或`runtime`来启用动态映射。

### 动态字段映射

当Elasticsearch检测到文档中出现新字段时，会根据当前配置动态参数为true或者runtime的不同，按照下表规则映射为不同的数据类型。

| JSON数据类型                       | dynamic: true                  | dynamic: runtime               |
| :----------------------------------: | :------------------------------: | :------------------------------: |
| null                               | 不添加字段                     | 不添加字段                     |
| true或者false                      | boolean                        | boolean                        |
| double                             | float                          | double                         |
| integer                            | long                           | long                           |
| object                             | object                         | object                         |
| array                              | 取决于数组中第一个非空值的类型 | 取决于数组中第一个非空值的类型 |
| 通过日期检测的string               | date                           | date                           |
| 通过数字检测的string               | float或者long                  | double或者long                 |
| 没有通过日期检测与数字检测的string | 带有.keyword子域的text         | keyword                        |

若将dynamic设置为false，出现新的字段，则会拒绝该文档。

### 日期检测

需设置date_detection属性为true（默认）；若设置为false，则字段会被映射为文本类型

```
PUT my-index-000001
{
  "mappings": {
    "date_detection": true
  }
}
```

默认值为[`"strict_date_optional_time"`,`"yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z"`]，满足该表达式的字段将被映射为日期类型（`strict_date_optional_time`为是一个通用的ISO日期时间分析器，至少包含年份，时间可选，日期与时间之间使用'T'分割）

#### 自定义检测格式

修改`dynamic_date_formats`属性，添加或删减规则

```console
PUT my-index-000001
{
  "mappings": {
    "dynamic_date_formats": ["strict_date_optional_time","yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z","MM/dd/yyyy"]
  }
}
```

### 数组检测

默认禁用，可以通过设置来启用

```
PUT my-index-000001
{
  "mappings": {
    "numeric_detection": true
  }
}
```

该检测将自动区分整形、浮点型和字符

## 动态模板

使用动态模板可以在默认的规则之外自定义映射规则。

```json
  "dynamic_templates": [
    {
      "模板名称": {
        ... 匹配条件 ... 
        "mapping": { ... } 
      }
    },
    ...
  ]
```

- `match_mapping_type`匹配Elasticsearch检测到的数据类型。
- `match`和`unmatch`使用一个模式来匹配字段名。
- `path_match`和`path_unmatch`对字段的完整点状路径进行操作。

模板是按照定义顺序处理的，当匹配到第一个匹配模板时将不再继续。

## 显式映射

