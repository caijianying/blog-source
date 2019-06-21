---
title: kafka patition计算
date: 2017-03-05 20:50:42
tags: 大数据
categories:
- kafka
---
# kafka patition计算
kafka生产信息的时候，可以指定该信息存储在具体的patition上，如果不指定会使用默认的算法计算要落的patition.以下是针对此做一个探讨和理解

## 消息路由
1. 指定了 patition，则直接使用；
2. 未指定 patition 但指定 key，通过对 key 的 value 进行hash 选出一个 patition
3. patition 和 key 都未指定，使用轮询选出一个 patition。

## Kafka Client计算patition
详细代码如下：
```java
//创建消息实例
public ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value) {
     if (topic == null)
          throw new IllegalArgumentException("Topic cannot be null");
     if (timestamp != null && timestamp < 0)
          throw new IllegalArgumentException("Invalid timestamp " + timestamp);
     this.topic = topic;
     this.partition = partition;
     this.key = key;
     this.value = value;
     this.timestamp = timestamp;
}

//计算 patition，如果指定了 patition 则直接使用，否则使用 key 计算
private int partition(ProducerRecord<K, V> record, byte[] serializedKey , byte[] serializedValue, Cluster cluster) {
     Integer partition = record.partition();
     if (partition != null) {
          List<PartitionInfo> partitions = cluster.partitionsForTopic(record.topic());
          int lastPartition = partitions.size() - 1;
          if (partition < 0 || partition > lastPartition) {
               throw new IllegalArgumentException(String.format("Invalid partition given with record: %d is not in the range [0...%d].", partition, lastPartition));
          }
          return partition;
     }
     return this.partitioner.partition(record.topic(), record.key(), serializedKey, record.value(), serializedValue, cluster);
}

// 使用 key 选取 patition
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
     List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
     int numPartitions = partitions.size();
     if (keyBytes == null) {
          int nextValue = counter.getAndIncrement();
          List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
          if (availablePartitions.size() > 0) {
               int part = DefaultPartitioner.toPositive(nextValue) % availablePartitions.size();
               return availablePartitions.get(part).partition();
          } else {
               return DefaultPartitioner.toPositive(nextValue) % numPartitions;
          }
     } else {
          //对 keyBytes 进行 hash 选出一个 patition
          return DefaultPartitioner.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
     }
}
```
