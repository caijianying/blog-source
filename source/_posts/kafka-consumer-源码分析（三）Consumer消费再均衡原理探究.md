---
title: kafka consumer 源码分析（三）Consumer消费再均衡原理探究
tags:
  - kafka
  - 专栏
originContent: ''
categories:
  - kafka
toc: false
date: 2019-06-30 20:26:34
---

## 开篇

在开始这篇之前，先抛出问题，这章主要通过研究consumer源码解决如下问题：

消费再均衡的原理是什么？

## 正文

### 消费再均衡的原理

主要分为四步

**1.FIND_COORDINATOR**

根据`hash(group_id)%consumerOffsetPartitionNum`查找出对应的partition,再查找出该partitiom对应的leader所在的broker,即可获得GroupCoordinator

**2.JOIN_GROUP**

在这一步主要完成消费组leader选举（获取第一个加入的组为leader，如果没有,选择map中的第一个node）和分区分配策略

**3.SYNC_GROUP**

客户端向GroupCoordinator发起同步请求，获取步骤2的分区分配方案。

**4.HEARTBEAT**

GroupCoordinator通过心跳来确定从属关系。

## 参考

略
