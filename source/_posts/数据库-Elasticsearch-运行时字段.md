---
title: 数据库-Elasticsearch-运行时字段
date: 2021-07-10 20:41:54
tags: [数据库, Elasticsearch]
---

# 运行时字段（Runtime fields）

## 简介

运行时字段允许在未定义的情况下，随时添加到文档中，且无需进行重新索引。

### 优点

运行时字段不会被索引，所以添加运行时字段不会增加索引的大小。在提高速度的同时减少了存储空间占用。

运行时字段可以随时添加到文档中。这样不必提前解析数据并定义文档结构。

### 缺点

运行时字段会影响搜索性能

### 关闭运行时字段的查询

设置`search.allow_expensive_queries`为`false`

### 添加运行时字段

在定义映射时添加一个`runtime`部分来映射运行时字段

```console
PUT my-index/
{
  "mappings": {
    "runtime": {
      "day_of_week": {
        "type": "keyword",
        "script": {
          "source": "emit(doc['@timestamp'].value.dayOfWeekEnum.getDisplayName(TextStyle.FULL, Locale.ROOT))"
        }
      }
    },
    "properties": {
      "@timestamp": {"type": "date"}
    }
  }
}
```

运行时字段支持的数据类型:

- boolean
- date
- double
- geo_point
- ip
- keyword
- long

若映射中设置了动态参数为`runtime`，则新添加的字段会自动被映射为运行时字段

```console
PUT my-index
{
  "mappings": {
    "dynamic": "runtime",
    "properties": {
      "@timestamp": {
        "type": "date"
      }
    }
  }
}
```

### 修改或删除运行时字段

```
PUT my-index/_mapping
{
 "runtime": {
   "day_of_week": null
 }
}
```

## 在查询请求中定义

在查询请求中通过添加`runtime_mappings`部分来动态定义一个运行时字段，仅在当前请求中有效。

```
GET my-index/_search
{
  "runtime_mappings": {
    "day_of_week": {
      "type": "keyword",
      "script": {
        "source": "emit(doc['@timestamp'].value.dayOfWeekEnum.getDisplayName(TextStyle.FULL, Locale.ROOT))"
      }
    }
  },
  "aggs": {
    "day_of_week": {
      "terms": {
        "field": "day_of_week"
      }
    }
  }
}
```

### 覆盖运行时字段类型

当部分数据被映射为非预期的或者错误的数据类型时，可以使用运行时字段来覆盖已经被映射的字段。

如下`measures.start`和`measures.end`原来被映射为`text`类型，使用运行时字段覆盖其映射类型，便可以使用long类型的特殊函数或者操作（如聚合等）。

```console
PUT my-index/_mapping
{
  "runtime": {
    "measures.start": {
      "type": "long"
    },
    "measures.end": {
      "type": "long"
    }
  }
}
```

### 定义一个不存在的字段

可以定义一个不存在的字段，并在同一个请求中对其进行统计汇总。

```console
GET my-index/_search
{
  "runtime_mappings": {
    "duration": {
      "type": "long",
      "script": {
        "source": """
          emit(doc['measures.end'].value - doc['measures.start'].value);
          """
      }
    }
  },
  "aggs": {
    "duration_stats": {
      "stats": {
        "field": "duration"
      }
    }
  }
}
```

## 可以使用_search API查询运行时字段数据

```
GET my-index/_search
{
  "fields": [
    "@timestamp",
    "day_of_week"
  ],
  "_source": false
}
```

其中`day_of_week`不是原始数据，为单独定义的运行时字段

