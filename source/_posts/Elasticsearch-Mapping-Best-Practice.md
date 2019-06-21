---
title: Elasticsearch Mapping Best Practice
date: 2019-03-02 22:26:48
tags: 经验
categories:
- elasticsearch
---
## Mapping各字段的选型流程

![image](https://github.com/TrumanDu/pic_repository/blob/master/Mapping%E5%90%84%E5%AD%97%E6%AE%B5%E7%9A%84%E9%80%89%E5%9E%8B%E6%B5%81%E7%A8%8B.png?raw=true)

## 规则
elasticsearch 为了更好的让大家开箱即用，默认启用了大量不必要的设置，为了降低空间使用，提升查询效率，请按以下规则构建最佳的Mapping

### 1. 禁用不需要的功能
#### 不用查询，禁用index
```
PUT index
{
  "mappings": {
    "_doc": {
      "properties": {
        "foo": {
          "type": "integer",
          "index": false
        }
      }
    }
  }
}
```

#### 不关心评分，禁用该功能
```
PUT index
{
  "mappings": {
    "_doc": {
      "properties": {
        "foo": {
          "type": "text",
          "norms": false
        }
      }
    }
  }
}
```
#### 不需要短语查询，禁用index positions
```
PUT index
{
  "mappings": {
    "_doc": {
      "properties": {
        "foo": {
          "type": "text",
          "index_options": "freqs"
        }
      }
    }
  }
}
```

### 2. 不要使用默认的Mapping
默认Mapping的字段类型是系统自动识别的。其中：string类型默认分成：text和keyword两种类型。如果你的业务中不需要分词、检索，仅需要精确匹配，仅设置为keyword即可。

根据业务需要选择合适的类型，有利于节省空间和提升精度。
```
PUT index
{
  "mappings": {
    "_doc": {
      "dynamic_templates": [
        {
          "strings": {
            "match_mapping_type": "string",
            "mapping": {
              "type": "keyword"
            }
          }
        }
      ]
    }
  }
}
```

### 3. 考虑identifiers映射为keyword

某些数据是数字的事实并不意味着它应该始终映射为数字字段。 Elasticsearch索引数字的方式可以优化范围查询，而keyword字段在term查询时更好。
通常，存储诸如ISBN的标识符或标识来自另一数据库的记录的任何数字的字段很少用于范围查询或聚合。这就是为什么他们可能会受益于被映射为keyword而不是integer或long。

## 参考
1. [Tune for disk usage](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-disk-usage.html#tune-for-disk-usage)
2. [让 Elasticsearch 飞起来：性能优化实践干货](https://toutiao.io/posts/lkqldx/preview)