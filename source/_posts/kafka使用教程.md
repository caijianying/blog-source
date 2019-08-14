---
title: kafka使用教程
date: 2017-03-05 20:38:06
tags: 大数据
categories:
- kafka
---
# kafka使用教程

## shell操作

**1.创建topic**

--replication-factor 2：备份因子为2

--partitions 10：partitions数目为10
 ```
bin/kafka-topics.sh --create --zookeeper 192.168.0.101:2181 --replication-factor 2 --partitions 10 --topic upgrade-kafka_test
 ```
**2.查询topic**

列出所有的topic
  ```
  bin/kafka-topics.sh --list --zookeeper localhost:2181
  ```
**3.删除topic**

首先需要确认集群是否配置delete.topic.enable=true，配置后即可删除，确保topic没有被使用。
```
bin/kafka-topics.sh --delete --zookeeper 192.168.0.101:2181 --topic upgrade-kafka_test

```

## 程序操作






## 参考
1. [kafka official website](https://kafka.apache.org/documentation/)
