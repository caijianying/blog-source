---
title: RedisCluster监控系统
date: 2016-10-24 21:35:24
tags: nosql
categories:
- redis
---
## 一、项目介绍
为了更好监控RedisCluster集群状态信息，提升性能，抛弃之前通过java api获取info信息。此次项目分为两个方面：

1. 通过redis自带info信息监控集群
2. 监控网络流量获取对集群的使用情况的监控。

总之，通过不同的粒度监控RedisCluster集群运行状况，提供良好的管理运维平台。

## 二、技术架构
1.info信息


```
graph LR
Logstash-->ElasticSearch
ElasticSearch-->Kibana
```
Logstash良好的插件结构设计，我们可以根据不同场景选择合适的input,filter,output插件。为了高效配置监控集群，input插件我们基于exec自定义了自己的插件redis-exec插件。output插件直接选择elasticsearch插件。

2.网络流量

```
graph LR
Packagebeat-->ElasticSearch
ElasticSearch-->Kibana
```
Packagebeat是一个分布式网络数据抓包软件，可以直接监控redis协议信息。

Kibana是一个es可视化的作图工具，根据不同的搜索条件可制定监控不同指标信息的动态图，让使用人员可以直观监控集群运行状况。
## 三、难点
1.redis-exec插件开发

详见logstash插件开发
2.监控指标

[详见国外案例](http://logz.io/blog/redis-performance-monitoring-elk-stack/)