---
title: Redis维护总结
date: 2016-06-01 21:58:44
tags: 大数据
categories:
- redis
---
#### 内存淘汰
---
获取内存淘汰配置策略
```
127.0.0.1:6379>config get maxmemory-policy
```
- volatile-lru：使用LRU算法从已设置过期时间的数据集合中淘汰数据。
- volatile-ttl：从已设置过期时间的数据集合中挑选即将过期的数据淘汰。
- volatile-random：从已设置过期时间的数据集合中随机挑选数据淘汰。
- allkeys-lru：使用LRU算法从所有数据集合中淘汰数据。
- allkeys-random：从数据集合中任意选择数据淘汰
- no-enviction：禁止淘汰数据。

#### 延时追踪及慢查询
---
Redis的延迟数据是无法从info信息中获取的。倘若想要查看延迟时间，可以用	Redis-cli工具加--latency参数运行，如:

```
Redis-cli --latency -h 127.0.0.1 -p 6379
```

结果为查询ID、发生时间、运行时长和原命令 默认10毫秒，默认只保留最后的128条。单线程的模型下，一个请求占掉10毫秒是件大事情，注意设置和显示的单位为微秒，注意这个时间是不包含网络延迟的。
```
127.0.0.1:6379>slowlog get
```
查看aof是否影响延迟,在info Stats中查看，数字代表有多少次fork延迟操作
```
latest_fork_usec:1724##单位微妙

```
查看最近一次fork延迟耗时
```
aof_delayed_fsync:0###被延迟的 fsync 调用数量
```
#### 主从切换
在运维过程中，整体断电后，集群重启后，master与slave节点分布可能会出现变化，为了保证集群master分布均匀，可使用命令主动切换master.
在需要提升为master的节点执行以下命令
```
127.0.0.1:6379>cluster failover
```
#### 指定slave数量
可在redis.conf中配置
```
cluster-migration-barrier 1
```
该参数指定slave最小数量为1，当前节点无法满足的话，会从别的节点漂移一个作为master的slave
