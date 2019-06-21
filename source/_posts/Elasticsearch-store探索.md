---
title: Elasticsearch store探索
date: 2019-03-02 22:24:31
tags: research
categories:
- elasticsearch
---

## store是什么？

回答store是什么之前，先说一下正常使用es,我们的字段默认store:false,但是我们还是可以正常查出该数据的，那store:true有什么用呢？

默认情况下，字段值(index:true)以使其可搜索，但不会存储它们。这意味着可以查询该字段，但无法检索原始字段值。

通常这没关系。字段值已经是_source字段的一部分，默认情况下存储该字段。如果您只想检索单个字段或几个字段的值，而不是整个_source，则可以使用[source filtering](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-source-filtering.html)来实现。



现在我来说一下store是什么？字面意思是是否存储该数据，如果为true，则会**单独存储该数据。如果_source 没有排除 exclude 掉这个字段，那么应该是会存储多份的。**

如果你要求返回field1（store：true），es会分辨出field1已经被存储了，因此不会从_source中加载，而是从field1的存储块中加载

## 意义
当数据有title, date,a very large content，而只需要取回title, date，不用从很大的_source字段中抽取。这种场景下性能更高。

## 怎么使用？
```
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "title": {
          "type": "text",
          "store": true 
        },
        "date": {
          "type": "date",
          "store": true 
        },
        "content": {
          "type": "text"
        }
      }
    }
  }
}

PUT my_index/_doc/1
{
  "title":   "Some short title",
  "date":    "2015-01-01",
  "content": "A very long content field..."
}

GET my_index/_search
{
  "stored_fields": [ "title", "date" ] 
}

```


## 相关原理


当你将一个field的store属性设置为true，这个会在lucene层面处理。lucene是倒排索引，可以执行快速的全文检索，返回符合检索条 件的文档id列表。在全文索引之外，lucene也提供了存储字段的值的特性，以支持提供id的查询（根据id得到原始信息）。通常我们在lucene层 面存储的field的值是跟随search请求一起返回的（id+field的值）。es并不需要存储你想返回的每一个field的值，因为默认情况下每 一个文档的的完整信息都已经存储了，因此可以跟随查询结构返回你想要的所有field值。 

有一些情况下，显式的存储某些field的值是必须的：当_source被disabled的时候，或者你并不想从source中parser来得到 field的值（即使这个过程是自动的）。

请记住：**从每一个stored field中获取值都需要一次磁盘io，如果想获取多个field的值，就需要多次磁盘io，但是，如果从_source中获取多个field的值，则只 需要一次磁盘io，因为_source只是一个字段而已**。所以在大多数情况下，从_source中获取是快速而高效的。 


es中默认的设置_source是enable的，存储整个文档的值。这意味着在执行search操作的时候可以返回整个文档的信息。如果不想返回这个文 档的完整信息，也可以指定要求返回的field，es会自动从_source中抽取出指定field的值返回（比如说highlighting的需求）。

## 注意事项
哪些情形下需要显式的指定store属性呢？大多数情况并不是必须的。从_source中获取值是快速而且高效的。如果你的文档长度很长，存储 _source或者从_source中获取field的代价很大，

你可以显式的将某些field的store属性设置为true。缺点如上边所说：假设你存 储了10个field，而如果想获取这10个field的值，则需要多次的io，如果从_source中获取则只需要一次，而且_source是被压缩过 的

## 引用
1. [mapping-store](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-store.html#mapping-store)
2. [elasticsearch的store属性 vs _source字段](https://www.cnblogs.com/Hai--D/p/5761525.html)
