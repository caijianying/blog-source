---
title: Elasticsearch使用备注
date: 2016-10-24 21:17:55
tags: research
categories:
- elasticsearch
---
# Elasticsearch使用备注
## 简介
beats + elasticsearch +logstash + kibana 这套工具集合出自于Elastic公司   https://www.elastic.co/guide/index.html
工具集功能
- beats是结合elasticsearch,logstash,kibana进行数据分析展示工具集
- beats主动获取数据，如：日志数据，文件数据，top数据，网络包数据，数据库数据等
- logstash（可选）用于日志分析，然后将分析后的数据存储到elasticsearch中
- elasticsearch用于分析、存储beats获取的数据
 - kibana用于展示图形elasticsearch上的数据，如：线图，饼图，表格等
 
 ## 备注
1. Elasticsearch rest api
- 查询模板
```
http://localhost:9200/_template
```
- 查询索引
```
http://localhost:9200/_cat/indices
```
- 查重指定索引数据
```
http://localhost:9200/packetbeat-*/_search?pretty
```
2. Kibana地址
```
http://localhost:5601/
```
3. Plugin Head集群可视化管理工具
需要额外安装
访问地址如下：
```
http://localhost:9200/_plugin/head/
```

## index template管理
1.删除模板
```
curl -XDELETE 'http://localhost:9200/_template/packetbeat'
```
2.上传模板
```
curl -XPUT 'http://localhost:9200/_template/packetbeat' -d@/etc/packetbeat/packetbeat.template.json
```
3.删除documents
```
curl -XDELETE 'http://localhost:9200/packetbeat-*'
```
