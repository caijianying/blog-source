---
title: Newegg的事件中心之Event Hub
tags: []
originContent: ''
categories:
  - 技术分享
toc: false
date: 2021-02-04 21:51:54
---



# Newegg的事件中心之Event Hub

## 万事皆事件

事件是信息的一种承载媒介，描述特定对象某一瞬间的非持续性变化，与唯一时刻和唯一对象关联。例如：某台计算机从运行状态变更为关机，程序运行开始和结束，办公大楼停电等。事件是对象在两个不同状态中的变更瞬间的记录。

对于事件，我们关注时间点，什么事件，什么状态。在企业中存在大量的事件，系统事件，监控事件，业务事件等，通过对事件的治理和挖掘，能够发现很多价值，解决切实的痛点。基于以上思考，我们构建了Event Hub。

## Event Hub简介

Event Hub是一个高度可缩放、分布式、基于时间序列的事件中心，能够实时的处理流式事件并进行告警和提醒。

Event Hub作为Newegg事件信息中枢，产品化新蛋各**产品资源**及平台**底层基础设施服务**生命周期与运转中的重要事件信息，并构建完善的事件消费渠道与流程，支撑线上监控与运维。

Event Hub产品化提供的事件信息，由Newegg内部各产品模块与底层基础设施服务获取，经过聚合，判定和收敛再最终呈现。信息源来自各模块底层的系统日志与监控项，保障客户透传客户的信息准确性与价值。

![](https://mc.qcloudimg.com/static/img/07e6540c12a045232174cdf3646e91ff/image.png)

## 目前应用场景

### 企业级监控/告警平台

在Event Hub之前公司监控存在一些问题：

- 告警不可追溯
- 告警不可指派
- 状态可变更很弱
- 监控信息可视化很弱
- 没有更好的统计报表

为了解决以上问题，治理企业级监控问题，我们在Event Hub中基于现存的问题，构建了企业级监控平台。俗话说，先挑软柿子捏。

作为企业级监控平台，Event Hub 立足于能够助力发现、定位、解决问题，保障系统与服务整体的稳定与性能。引入事件作为监控的信息载体，能更准确与直接描述**资源**与**底层基础设施服务**的运行状态，助力更高效发现、定位从而解决问题。致力于提交信息描述准确性，减少延迟，传递更多的信息，完善监控信息维度，使用通用事件引擎对告警类信息加工处理，尽而告警。

![1611048694361](http://blogstatic.aibibang.com/1611048694361.png)

####  不仅仅是一个告警，而是所有的告警

Event Hub不仅能够接入来自底层基础设施服务例如：Syslog, SNMP, Prometheus, Nagios, Zabbix, Sensu 和 netdata。任何监控工具和系统都可以很容易通过URL方式将告警信息的集成。接入成本几乎为零，降低各个系统接入的难度，统一一个平台，在一个平台上管理很查看不同的告警。

![image-20210204214604410](http://blogstatic.aibibang.com/image-20210204214604410.png)

#### 灵活的警报格式。记录对你重要的事情

大多数的监控系统强制你按照它的格式去做，但是Event Hub不同，你可以自由发送任何值的警报。单个警报可以与多个服务关联，具有任何格式的任意数量的“标签”，并且允许任意数量的自定义属性。

```
{
  "severity": "major",
  "eventId": "714f866ba2ec43775cd6bbda65ff65bd6091bc3a26e9f9f61045014e7746dc09",
  "resource": "5XX Error",
  "origin": "API Gateway",
  "eventType": "alert",
  "type": "apigatewayAlert",
  "environment": "EC",
  "createTime": "2021-01-20T01:44:10.894Z",
  "service": [
    "Desktop-Shopping"
  ],
  "postDate": 1611107052262,
  "attributes": {
    "apiName": "Desktop-Shopping",
    "owner-email": "........",
    "categoryName": "Website API (PCI)",
    "moreInfo": "<a href=\"https://xxxxxxxxx/xxxxxxx/apis/xxxxxxxxxxxx/monitor?env=prd&location=e4\">API Gateway Monitor</a>",
    "apiId": "xxxxxxxxxxxxxxx",
    "categoryId": "xxxxxxxxxxxxxxxxxxxxxxxx"
  },
  "location": "E4",
  "text": "51 5XX calls in the past 5 minutes. ",
  "event": "API (Desktop-Shopping) Has 5XX Issue in prd e4",
  "value": 51,
  "status": "open",
  "timestamp": 1611107050996
}
```



![image-20210204214203000](http://blogstatic.aibibang.com/image-20210204214203000.png)

#### 零成本重复数据删除和简单关联

当同时收到多个来源接收警报时，你很快就会不知所措。对于Event Hub，如果警报和警报具有相同的严重性，则具有相同环境和资源的任何警报将被视为重复警报。

![image-20210204214411771](http://blogstatic.aibibang.com/image-20210204214411771.png)

#### 更智能的统计报表

![1611111265964](http://blogstatic.aibibang.com/1611111265964.png)



当然除了以上说了，还有很多地方，限于时间和篇幅，如果你的系统需要这么一个企业级监控，请联系我。

### 统一的信息通知/任务追踪平台

可以查看待完成任务/通知（事件）

支持多种信息发送方式：teams,email.webhook等

###  实现基于时间序列的事件驱动引擎

通用的时间序列事件引擎，支持对事件流式数据进行强大的过滤，数据加工，事件流转。

![img](http://blogstatic.aibibang.com/common_event_process.png)

## 系统设计

### 系统架构

![1611120821430](http://blogstatic.aibibang.com/1611120821430.png)

### 数据流

![img](http://blogstatic.aibibang.com/data_stream.png)

## 价值

- 完善资源监控信息维度，为监控运维提供更全面数据支撑。
- 提供事件信息消费渠道，助力转化监控信息价值。
- 收敛判定逻辑，更高效直接定位影响资源及致因。
- 事件信息可溯源审阅，资源及平台生命周期重要事件变更知悉。
- 事件触发联动，自动化响应特定资源及环境变更。

## 未来的路

未来还可基于平台中的事件流，做到事件流驱动应用：

- 反欺诈检测
- 异常检测，比如电商网站中，用户登录异常检测
- 企业内部安全检测