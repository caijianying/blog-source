---
title: Redis Cluster Resharding
date: 2016-08-21 20:27:06
tags: 大数据
categories:
- redis
---
# Redis Cluster Resharding实践
## 简介
在Redis Cluster运维过程中，会出现水平扩展集群，而水平扩展集群即新增master节点。Redis Cluster需要就需要重新划分slot，数据迁移等操作，本文只是探讨实现过程，用Redis-cli自带命令实现Resharding。
## 实践
### 过程简介
真正开始Resharding之前，会先在源结点和目的结点上执行cluster setslot <slot> importing和cluster setslot <slot> migrating命令，将要迁移的槽分别标记为迁出中和导入中的状态。然后，执行cluster getkeysinslot获得Slot中的所有Key。最后就可以对每个Key执行migrate命令进行迁移了。槽迁移完成后，执行cluster setslot命令通知整个集群槽的指派已经发生变化。

关于迁移过程中的数据访问，客户端访问源结点时，如果Key还在源结点上就直接操作。如果已经不在源结点了，就向客户端返回一个ASK错误，将客户端重定向到目的结点。
### 实践过程
#### 1.redis-cli 实现
迁移备注：
实现迁移9013node中952 slot到9015 node上,其中源9013 node_id为a92e4554aa2828b85f50f8f8318429d68f5213ca，目的9015node_id为c725cda7b314701c6892f035224700e9a8336699
- 详解
```
在迁移目的节点执行cluster setslot <slot> IMPORTING <node ID>命令，指明需要迁移的slot和迁移源节点。
在迁移源节点执行cluster setslot <slot> MIGRATING <node ID>命令，指明需要迁移的slot和迁移目的节点。
在迁移源节点执行cluster getkeysinslot获取该slot的key列表。
在迁移源节点执行对每个key执行migrate命令，该命令会同步把该key迁移到目的节点。
在迁移源节点反复执行cluster getkeysinslot命令，直到该slot的列表为空。
在迁移源节点或者目的节点执行cluster setslot <slot> NODE <node ID>，完成迁移操作。
```
- 实践
```
redis-cli -c -p 9015 cluster  setslot 952 importing a92e4554aa2828b85f50f8f8318429d68f5213ca//目的节点执行
redis-cli -c -p 9013 cluster  setslot 952 migrating c725cda7b314701c6892f035224700e9a8336699//源节点执行
redis-cli -c -h 192.168.0.101 -p 9013 migrate 192.168.0.101 9015 "truman:00000829" 0 1000
redis-cli -c -p 9015 cluster  setslot 952  node c725cda7b314701c6892f035224700e9a8336699
```
相关命令参考如下：
```
//集群
CLUSTER INFO 打印集群的信息
CLUSTER NODES 列出集群当前已知的所有节点（node），以及这些节点的相关信息。
//节点
CLUSTER MEET <ip> <port> 将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。
CLUSTER FORGET <node_id> 从集群中移除 node_id 指定的节点。
CLUSTER REPLICATE <node_id> 将当前节点设置为 node_id 指定的节点的从节点。
CLUSTER SAVECONFIG 将节点的配置文件保存到硬盘里面。
//槽(slot)
CLUSTER ADDSLOTS <slot> [slot ...] 将一个或多个槽（slot）指派（assign）给当前节点。
CLUSTER DELSLOTS <slot> [slot ...] 移除一个或多个槽对当前节点的指派。
CLUSTER FLUSHSLOTS 移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点。
CLUSTER SETSLOT <slot> NODE <node_id> 将槽 slot 指派给 node_id 指定的节点，如果槽已经指派给另一个节点，那么先让另一个节点删除该槽>，然后再进行指派。
CLUSTER SETSLOT <slot> MIGRATING <node_id> 将本节点的槽 slot 迁移到 node_id 指定的节点中。
CLUSTER SETSLOT <slot> IMPORTING <node_id> 从 node_id 指定的节点中导入槽 slot 到本节点。
CLUSTER SETSLOT <slot> STABLE 取消对槽 slot 的导入（import）或者迁移（migrate）。
//键
CLUSTER KEYSLOT <key> 计算键 key 应该被放置在哪个槽上。
CLUSTER COUNTKEYSINSLOT <slot> 返回槽 slot 目前包含的键值对数量。
CLUSTER GETKEYSINSLOT <slot> <count> 返回 count 个 slot 槽中的键。
```
#### 2.redis-trib.rb 实现
```
#把127.0.0.1:9013当前master迁移到127.0.0.1:9015上  
redis-trib.rb reshard 127.0.0.1:9013  
#根据提示选择要迁移的slot数量(ps:这里选择500)  
How many slots do you want to move (from 1 to 16384)? 1(源master的需要迁移slot数量，不能单个指定)  
#选择要接受这些slot的node-id(127.0.0.1:9015)  
What is the receiving node ID? c725cda7b314701c6892f035224700e9a8336699 (ps:127.0.0.1:9015的node-id)  
Please enter all the source node IDs.  
  Type 'all' to use all the nodes as source nodes for the hash slots.  
  Type 'done' once you entered all the source nodes IDs.  
Source node #1:a92e4554aa2828b85f50f8f8318429d68f5213ca(源master的node-id)  
Source node #2:done  
#打印被移动的slot后，输入yes开始移动slot以及对应的数据.  
#Do you want to proceed with the proposed reshard plan (yes/no)? yes  
```
## 参考
1. http://redis.io/commands/cluster-setslot
2. http://carlosfu.iteye.com/blog/2243056（该博客较详细，本文未参考，只是做记录）
