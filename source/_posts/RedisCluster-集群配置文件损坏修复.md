---
title: RedisCluster 集群配置文件损坏修复
date: 2017-06-01 21:15:33
tags: 大数据
categories:
- redis
---
# RedisCluster 集群配置文件损坏修复

## 问题现象

启动集群后，log文件显示如下信息：
```
Unrecoverable error: corrupted cluster config file.

```
这种情况，因为集群cluster-config-file文件损坏引起，导致该节点无法启动

## 修复方案

1. 首先在各个node上移除该出错节点
2. 删除该cluster-config-file文件
3. 重新启动该节点
4. 将该节点加入集群
5. 指定该节点的master,将该节点以slave加入集群(本次修复未执行，集群自动恢复正常)



##  实施步骤

**一、首先在各个node上移除该出错节点**

执行以下脚本，将fail节点移除出集群,该操作仅移除一个机器中的无用节点信息，如果是多个机器，请在host_array中添加多个机器IP
```
#!/bin/bash
host_array=(192.168.0.101)
  for var in  ${host_array[@]};
  do
          for i in $(seq 7000 7012)
          do
             nodeids=`src/redis-cli -c -h $var -p $i cluster nodes|grep fail|awk '{print $1}'`
              for d in $nodeids
              do
                    echo $d
                    src/redis-cli -c -h $var -p $i cluster forget $d
              done
          done
  done
```

**二、删除该cluster-config-file文件**
```
 mv nodes-7004.conf nodes-7004.conf.bak
```

**三、重新启动该节点**

```
redis-server redis.conf
```


**四、将该节点加入集群**

在集群任意instance 执行以下命令
```
CLUSTER MEET <ip> <port>   //将ip和port所指定的节点添加到集群当中，让它成为集群的一份子
```


**五、指定该节点的master,将该节点以slave加入集群**

**在此次修复场景中未进行第五步，集群自动分配成为不均衡节点的slave**

如果集群未自动分配，在需要加入集群的实例上执行，指定为某一个master的slave
```
CLUSTER REPLICATE <node_id> //将当前节点设置为 node_id 指定的节点的从节点。
```
