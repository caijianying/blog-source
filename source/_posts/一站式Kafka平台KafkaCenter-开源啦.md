---
title: 一站式Kafka平台KafkaCenter-开源啦
tags:
  - kafka
categories:
  - kafka
toc: false
date: 2020-03-26 22:11:46
---

# 一站式Kafka平台KafkaCenter-开源啦
![](http://blogstatic.aibibang.com/screenshot.png)

**Important: [https://github.com/xaecbd/KafkaCenter](https://github.com/xaecbd/KafkaCenter)**
## 前言
经过一年的不断打磨，在团队成员的共同努力下，终于能以真实的面貌呈现在大家的面前，很开心，很激动。开源软件，只是为了和**大家交个朋友**,喜欢的话，**star，star，star**,重要的事情说三遍！

![](http://blogstatic.aibibang.com/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

之前做过[Kafka 平台化的一点经验分享](https://mp.weixin.qq.com/s/C6qIg9H6Og3AHzXJq9AYdQ)，以至于很多小伙伴问了，这个东西有没有开源，在团队成员的共同努力下，欢迎感兴趣的同学加入我们，做点感兴趣的事。

![](http://blogstatic.aibibang.com/kafka-center.png)

## KafkaCenter是什么？
KafkaCenter是Kafka 集群管理和维护，生产/消费监控，生态组件使用的统一一站式平台。
## KafkaCenter 解决了什么问题
在给大家说我们解决什么问题之前，先说说在没有KafkaCenter之前我们的面临的问题。
### 我们面临的问题
- 创建topic，人工处理化
- 相关kafka运维，监控孤岛化
- 现有消费监控工具监控不准确
- 无法拿到Kafka 集群的summay信息
- 无法快速知晓集群健康状态
- 无法知晓业务对team kafka使用情况
- kafka管理，监控工具稀少，没有一个好的工具我们直接可以使用
- 无法快速查询topic消息
### Kafka Center解决了哪些问题
- **统一:** 一个平台，一站式包含自助，管理，监控，运维，使用一体化。
- **流程化:** 创建topic流程化，做到对topic使用全生命周期管理。
- **复用:** 平台支持接入多个集群，复用性很高。
- **成本:** 只用部署一套程序，节省机器资源。降低运维成本，高效运维。
- **生态:** 目前已经接入connect，ksql。
- **便捷:** 提供便捷工具，让无需有kafka使用经验的人，都可以方便生产、消费消息。
- **全局:** 可以站在不同的维度查看目前kafka使用情况
- **权限:** 完善的权限设计，减少风险漏洞。

## 功能模块介绍
- **Home**->
查看平台管理的Kafka Cluster集群信息及监控信息
- **Topic**->
用户可以在此模块查看自己的Topic，发起申请新建Topic，同时可以对Topic进行生产消费测试。
- **Monitor**->
用户可以在此模块中可以查看Topic的生产以及消费情况，同时可以针对消费延迟情况设置预警信息。
- **Kafka Connect**->
实现用户快速创建自己的Connect Job，并对自己的Connect进行维护。
- **KSQL**->
实现用户快速创建自己的KSQL Job，并对自己的Job进行维护。
- **Approve**->
此模块主要用于当普通用户申请创建Topic，管理员进行审批操作。
- **Setting**->
此模块主要功能为管理员维护User、Team以及kafka cluster信息
- **Kafka Manager**->
此模块用于管理员对集群的正常维护操作。

说了这么多，还是给大家看看主要系统截图吧！

![](http://blogstatic.aibibang.com/kafkamanager_clusterpng.png)
![](http://blogstatic.aibibang.com/cluster_monitor.png)
![](http://blogstatic.aibibang.com/consumer_alert.png)
![](http://blogstatic.aibibang.com/ksql_console.png)
![](http://blogstatic.aibibang.com/monitor_producer.png)
![](http://blogstatic.aibibang.com/monitor_consumer.png)
![](http://blogstatic.aibibang.com/monitor_chart.png)
![](http://blogstatic.aibibang.com/connect1.png)