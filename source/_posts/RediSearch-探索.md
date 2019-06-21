---
title: RediSearch 探索
date: 2017-07-19 22:17:29
tags: nosql
categories:
- redis
---
# RediSearch 探索

## 简介

RediSearch是一个高性能的全文搜索引擎，可作为一个Redis Module 运行在Redis上，是由RedisLabs团队开发的。

**特性**
- 多字段全文检索
- 增量索引无性能损失
- 文档排序
- 复杂的子查询 and,or,not
- 可选查询子句。
- 基于前缀的搜索
- 字段权重
- 自动完成建议（使用模糊前缀建议）
- 精确词组搜索，基于Slop的搜索
- 基于Stemming的查询扩展在许多语言（使用Snowball）
- 支持用于查询扩展和评分的自定义函数（请参阅扩展）。
- 限制搜索到特定文档字段（最多支持8个字段）。
- 数字过滤以及范围查找
- 使用Redis自己的地理命令进行地理过滤
- 支持任何utf-8编码文本
- 检索完整的文档内容或只是ids
- 将现有的HASH键自动索引为文档
- 使用索引垃圾回收文档删除和更新

## 入门

### 安装/运行
```
wget https://github.com/RedisLabsModules/RediSearch/archive/v0.19.1.tar.gz
tar xvf v0.19.1.tar.gz
cd RediSearch-0.19.1/src
make all
nohup redis-server --loadmodule ./redisearch.so &
```
**1. 创建索引**

```
127.0.0.1:6379> FT.CREATE myIDs SCHEMA title TEXT WEIGHT 5.0 body TEXT url TEXT
OK

```
**2. 增加文档到该索引**
```
127.0.0.1:6379> FT.ADD myIDs doc1 1.0 FIELDS title "hello world" body "lorem ipsum" url "http://redis.io"
OK

```

**3. 在该索引中搜索**
```
127.0.0.1:6379> ft.search myIDs "hello world" limit 0 10
1) (integer) 1
2) "doc1"
3) 1) "title"
   2) "hello world"
   3) "body"
   4) "lorem ipsum"
   5) "url"
   6) "http://redis.io"

```

**4. 删除索引**

```
127.0.0.1:6379> ft.drop myIDs
OK

```

**5. 添加并获取自动完成建议**
```
127.0.0.1:6379> ft.sugadd autocomplete "hello truman" 100
(integer) 1
127.0.0.1:6379> ft.sugget autocomplete "he"
1) "hello truman"

```


## 引用
1. [redisearch.io](#http://redisearch.io/)
