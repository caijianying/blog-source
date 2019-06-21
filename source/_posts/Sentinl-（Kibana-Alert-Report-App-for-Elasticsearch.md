---
title: Sentinl-（Kibana Alert & Report App for Elasticsearch)
date: 2017-09-17 12:24:28
tags: research
categories:
- elasticsearch
---
# Sentinl-(Kibana Alert&Report App for Elasticsearch)

## 前言
最近公司ES集群升级到5.5.1，新版增加了许多新的功能，也废弃了一些特性，同时整合一些插件，做了一个统一的封装，ES栈也越来越丰富，强大了。

最近有增加[X-PACK](https://www.elastic.co/guide/en/x-pack/current/xpack-introduction.html),X-Pack是一个Elastic栈扩展，将安全性，警报，监视，报告和图形功能捆绑到一个易于安装的软件包中。 X-Pack组件旨在无缝协同工作，您可以轻松地启用或禁用要使用的功能。

然并卵，这个东西是要收费的，但是这个并不妨碍开源方案，[Sentinl](https://github.com/sirensolutions/sentinl)就是一个开源方案，作为kibana插件，集成在kibana中，主要提供了预警和报告功能，在架构设计上往X-PACK上靠拢，只提供了一些基本功能，但对于目前一些简单业务需求，完全可以满足需求，这个软件开源不久，期待更多完善。
##  Sentinl简介
Sentinl 5扩展自Kibi / Kibana 5，具有警报和报告功能，可使用标准查询，可编程验证器和各种可配置操作来监控，通知和报告数据系列更改 - 将其视为一个独立的“观察者” “报告”功能（PNG / PDFs快照）。

SENTINEL还旨在通过直接在Kibana UI中整合来简化在Kibi / Kibana中创建和管理警报和报告的过程。


### 功能模块

- Watchers
- Alarms
- Reports


Watchers是Sentinl核心，主要由 input,Condition,Transform,Actions几大块组成，可以和X-Pack一一对应，部分文档可参考X-Pack，但需要注意的是它和X-Pack还有一些区别，主要体现在input只实现了search，其他并未实现，Actions也并未都实现

##  Sentinl安装与配置

**1. 安装**
```
/opt/kibana/bin/kibana-plugin install https://github.com/sirensolutions/sentinl/releases/download/tag-5.5/sentinl-v5.5.1.zip
```
**2. 配置**
在kibana.yml添加以下内容，可以根据具体需求删减
```
sentinl:
  es:
    timefield: '@timestamp'
    default_index: watcher
    type: watch
    alarm_index: watcher_alarms
  sentinl:
    history: 20
    results: 50
  settings:
    email:
      active: false
      user: username
      password: password
      host: smtp.server.com
      ssl: true
      timeout: 10000  # mail server connection timeout
    slack:
      active: false
      username: username
      hook: 'https://hooks.slack.com/services/<token>'
      channel: '#channel'
    report:
      active: false
      tmp_path: /tmp/
    pushapps:
      active: false
      api_key: '<pushapps API Key>'  
```
##  使用案例


**业务需求：**

监控指定索引1小时内数量大于1w,控制台提醒

**实践：**

1. 配置General

输入名称和监控频率
2. 配置input

```
{
  "search": {
    "request": {
      "index": [
        "shoppingcartapi-*"
      ],
      "body": {
        "_source": false,
        "query": {
          "bool": {
            "filter": {
              "range": {
                "post_date": {
                  "from": "now-1h"
                }
              }
            }
          }
        }
      }
    }
  }
}
```
其中shoppingcartapi-*为索引模糊名称，post_date为时间字段
3. 配置Condition

```
payload.hits.total > 10
```
4. 配置Action

配置一个console action message填写以下内容
```
payload.hits.total:{ {payload.hits.total} }
```
Tip:通过```{   { } }```可以拿出search结果中的值,此处有空格是避免hexo问题，正常使用无需如此。