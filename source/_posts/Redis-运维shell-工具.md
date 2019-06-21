---
title: Redis 运维shell 工具
date: 2017-07-19 22:17:15
tags: nosql
categories:
- redis
---
# Redis 运维shell 工具

## 介绍

在Redis 集群运维过程中，经常需要做一些重复性工作，因为Redis 无中心设计，这就需要在每个节点中执行相同命令，对于这些重复性劳动工作完全可以通过shell处理，降低运维难度，减少工作量，以下是我在工作冲总结的脚本，仅供大家参考。

## 工具集

### 1. 集群健康状态检测

新建脚本**redis_tool.sh**,然后执行

```
sh redis_tool.sh clusterStatus 127.0.0.1 7000
```

### 2. 移除fail节点

新建脚本**redis_tool.sh**,然后执行

```
sh redis_tool.sh forget 127.0.0.1 7000
```



### 脚本内容

```
#!/bin/sh

function clusterStatus(){
instans=`redis-cli -c -h $1 -p $2 cluster nodes|grep -v fail|awk '{print $2}'`
for var in  ${instans[@]};
do
         echo $var
         host=${var%:*}
         port=${var#*:}
	redis-cli -c -h $host -p $port cluster info |grep cluster_state:fail
done
}

function forget(){
instans=`redis-cli -c -h $1 -p $2 cluster nodes|grep -v fail|awk '{print $2}'`
for var in  ${instans[@]};
do
         echo $var
         host=${var%:*}
         port=${var#*:}
          failnodes=`redis-cli -c -h $host -p $port cluster nodes|grep  fail|awk '{print $1}'`
         for nodeid in  ${failnodes[@]};
         do  
             echo $nodeid
             redis-cli -c -h $host -p $port cluster forget $nodeid
         done

done
}

# main start
case $1 in
  clusterStatus)
    echo "check redis cluster cluster_state..."
    clusterStatus $2 $3;
  ;;
   forget)
    echo "forget redis fail nodes ..."
    forget $2 $3;
  ;;
  *)
    echo "Usage: the options [clusterStatus|forget]"
  ;;
esac
```
