---
title: kafka 逻辑架构设计
tags:
  - 专刊
  - kafka
originContent: ''
categories:
  - kafka
toc: false
date: 2019-07-08 16:30:16
---

# Kafka 逻辑架构设计

## 开篇

本文主要探讨如下问题；

1. Kafka架构设计
2. Kafka的日志目录结构

## 正文

### Kafka架构设计

kafka为分布式消息系统，由多个broker组成。消息是通过topic来分类的，一个topic下存在多个partition,每个partition又由多个segment构成。

#### 发布订阅者模式

![](http://blogstatic.aibibang.com/%E6%B6%88%E8%B4%B9%E8%AE%A2%E9%98%85%E8%80%85%E6%A8%A1%E5%BC%8F.jpg)

### kafka集群架构

![](http://blogstatic.aibibang.com/kafka%E9%9B%86%E7%BE%A4%E6%9E%B6%E6%9E%84.jpg)

### 主题逻辑结构

![](http://blogstatic.aibibang.com/Topic%E9%80%BB%E8%BE%91%E7%BB%93%E6%9E%84.jpg)

## Kafka的日志目录结构

![](http://blogstatic.aibibang.com/kfaka%E6%96%87%E4%BB%B6%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.jpg)

