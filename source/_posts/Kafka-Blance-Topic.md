---
title: Kafka Blance Topic
date: 2017-03-05 20:48:13
tags: 大数据
categories:
- kafka
---
# one step

创建topic-move-test.json,迁移可以指定多个topic

```
{"topics": [{
    "topic": "upgrade-kafka"
  },{
  "topic":"upgrade-kafka1"
  }],
  "version": 1
}
```
# two step
在kafka工作目录，执行以下生成迁移配置，其中91为需要迁移到的broker id
```
bin/kafka-reassign-partitions.sh --zookeeper 127.0.0.1:2181 --topics-to-move-json-file topic-move-test.json --broker-list "91" --generate
```
生成以下内容：
```
Current partition replica assignment

{"version":1,"partitions":[{"topic":"upgrade-kafka1","partition":1,"replicas":[96]},{"topic":"upgrade-kafka","partition":5,"replicas":[96]},{"topic":"upgrade-kafka1","partition":2,"replicas":[91]},{"topic":"upgrade-kafka","partition":1,"replicas":[92]},{"topic":"upgrade-kafka","partition":4,"replicas":[95]},{"topic":"upgrade-kafka1","partition":0,"replicas":[95]},{"topic":"upgrade-kafka","partition":3,"replicas":[94]},{"topic":"upgrade-kafka","partition":0,"replicas":[91]},{"topic":"upgrade-kafka","partition":2,"replicas":[93]}]}
Proposed partition reassignment configuration

{"version":1,"partitions":[{"topic":"upgrade-kafka","partition":5,"replicas":[91]},{"topic":"upgrade-kafka1","partition":1,"replicas":[91]},{"topic":"upgrade-kafka","partition":1,"replicas":[91]},{"topic":"upgrade-kafka","partition":4,"replicas":[91]},{"topic":"upgrade-kafka1","partition":2,"replicas":[91]},{"topic":"upgrade-kafka","partition":3,"replicas":[91]},{"topic":"upgrade-kafka1","partition":0,"replicas":[91]},{"topic":"upgrade-kafka","partition":0,"replicas":[91]},{"topic":"upgrade-kafka","partition":2,"replicas":[91]}]}

```
# three step

新建expand-cluster-reassignment.json文件，将上一步输出放入，内容如下：
```
{"version":1,"partitions":[{"topic":"upgrade-kafka","partition":5,"replicas":[91]},{"topic":"upgrade-kafka1","partition":1,"replicas":[91]},{"topic":"upgrade-kafka","partition":1,"replicas":[91]},{"topic":"upgrade-kafka","partition":4,"replicas":[91]},{"topic":"upgrade-kafka1","partition":2,"replicas":[91]},{"topic":"upgrade-kafka","partition":3,"replicas":[91]},{"topic":"upgrade-kafka1","partition":0,"replicas":[91]},{"topic":"upgrade-kafka","partition":0,"replicas":[91]},{"topic":"upgrade-kafka","partition":2,"replicas":[91]}]}
```
# four step
按自己需求可以重新编辑expand-cluster-reassignment.json内容，修改broker id.
执行迁移命令

```
bin/kafka-reassign-partitions.sh --zookeeper 127.0.0.1:2181 --reassignment-json-file expand-cluster-reassignment.json --execute
```
检查执行结果

```
bin/kafka-reassign-partitions.sh --zookeeper 127.0.0.1:2181 --reassignment-json-file expand-cluster-reassignment.json --verify
```

以下即为迁移成功：
```
Status of partition reassignment:
Reassignment of partition [upgrade-kafka,4] completed successfully
Reassignment of partition [upgrade-kafka,5] completed successfully
Reassignment of partition [upgrade-kafka1,0] completed successfully
Reassignment of partition [upgrade-kafka,2] completed successfully
Reassignment of partition [upgrade-kafka1,1] completed successfully
Reassignment of partition [upgrade-kafka,1] completed successfully
Reassignment of partition [upgrade-kafka,3] completed successfully
Reassignment of partition [upgrade-kafka,0] completed successfully
Reassignment of partition [upgrade-kafka1,2] completed successfully

```

# Reference
1. [kafka.apache.org](https://kafka.apache.org/documentation/#basic_ops_cluster_expansion)
