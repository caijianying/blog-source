---
title: Elasticsearch集群原理探索
date: 2018-07-22 18:40:23
tags: research
categories:
- elasticsearch
---
# Elasticsearch集群原理探索

## 1. Elasticsearch Master 选举过程

## 2. Elasticsearch 文档检索过程


## 3. Elasticsearch 分片分配

### 3.1 集群级别
在es集群，可以通过以下设置，控制分片分配进程。

- **Cluster Level Shard Allocation** 

集群级别分片分配 

在集群初始化(重启恢复)，副本分配，再平衡，或者节点加入、离开触发集群分配分片

- **Disk-based Shard Allocation** 

依据磁盘分片分配

磁盘因素（默认值85%）达到最低值，将阻止最新分片到该机器上，或者直接移除分片。

达到85%，阻止，达到90，将移除分片到其他机器 ，达到95%对该node上的index 强制只读，如果磁盘够用，撤销只读index block。

主要设置参数如下：
```
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.disk.watermark.low": "100gb",
    "cluster.routing.allocation.disk.watermark.high": "50gb",
    "cluster.routing.allocation.disk.watermark.flood_stage": "10gb",
    "cluster.info.update.interval": "1m"
  }
}
```
- **Shard Allocation Awareness and Forced Awarenessedit**

分片分配意识

分片分配感知设置允许您告知Elasticsearch您的硬件配置，主要是用来解决避免物理机划分多个虚拟带来不利影响。详细配置见[官网文档](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/allocation-awareness.html)

- **Shard Allocation Filtering**

主要存在以下参数

```
cluster.routing.allocation.include.{attribute}
```
分配分片到包含{attribute}node中
```
cluster.routing.allocation.require.{attribute}
```
分配分片到必须包含所有{attribute}node中
```
cluster.routing.allocation.exclude.{attribute}
```
分配分片到不包含{attribute}node中

attribute支持如下属性：


属性 | 注释
---|---
_name | 匹配节点名称
_ip| 匹配节点IP
_host|匹配hostnames



分片分配过滤

支持用include/exclude过滤器来控制分片的分配。过滤器可以设置在索引级别或者是集群级别

下线一个节点(10.0.0.1)可以按如下操作：
```
PUT _cluster/settings
{
  "transient" : {
    "cluster.routing.allocation.exclude._ip" : "10.0.0.1"
  }
}
```

除了逗号作为分隔列出多个值之外，所有属性值都可以用通配符指定

```
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.exclude._ip": "192.168.2.*"
  }
}
```

### 3.2 index级别

- **分片分配过滤** 控制哪些分片落在哪些节点

详见集群
- **延迟分配** 延迟因为一个节点离开分配未分配分片

1. 修改默认设置，默认值是1m

```
PUT _all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m"
  }
}

```
2. 监控延迟未分配分片

```
GET _cluster/health
```
3. 永久移除一个节点

如果一个节点要永久性移除，需要立即分配分片，仅仅需要将timeout 设置为0

```
PUT _all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "0"
  }
}
```

- **单个节点分片总数**  对来自每个节点的相同索引的碎片数量进行硬性限制

以下动态设置允许您为每个节点允许的单个索引指定分片总数的硬性限制
```
index.routing.allocation.total_shards_per_node
```


同样可以设置节点限制，不用关心index情况

```
cluster.routing.allocation.total_shards_per_node
```

以上大小都是包括主本与副本数量，两个默认值是无穷大



## 引用
1. [Elasticsearch分布式一致性原理剖析(一)-节点篇](https://zhuanlan.zhihu.com/p/34830403)
2. [elastic](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/modules-cluster.html)