layout: 'n'
title: RedisCluster搭建
date: 2016-07-18 21:15:05
tags: nosql
categories:
- redis
---
# RedisCluster搭建
## 前言
现在越来越多的公司开始使用redis cluster,这就要求大家能够学会快速搭建redis cluster集群，在官网上也是有很详细的入门指导，本文章只是自己搭建后总结，主要使用两种方式搭建：1. 官网上使用的redis-trib.rb脚本构建集群；2. 使用shel 命令搭建。

## 准备

因为两种方式都需要提前启动好相应的redis实例，因为将公共步骤放在准备工作中。这一步比较简单，主要是按端口新建不同文件夹，然后复制redis.conf,修改其中port对应的端口。集群高可用需要至少三个node,本次采用3 master、3slave架构。步骤如下：
1.
```
mkdir 9000
cp redis.conf 9000/
```
2.
修改复制后的redis.conf：
cluster-enabled yes
port 9000

3.
```
src/redis-server 9000/redis.conf
```
然后重复以上操作 9001-9005
## rb脚本创建
准备操作中已经建立了6个独立的node,接下来执行以下脚本：
```
/src/redis-trib.rb create --replicas 1 127.0.0.1:9000 127.0.0.1:9001 127.0.0.1:9002 127.0.0.1:9003 127.0.0.1:9004 127.0.0.1:9005
```
在执行脚本后，确认自动计算的分配方案，即可搭建好redis cluster集群
## shell创建
1.集群meet
```
src/redis-cli -p 9000 cluster meet 127.0.0.1 9001
src/redis-cli -p 9000 cluster meet 127.0.0.1 9002
src/redis-cli -p 9000 cluster meet 127.0.0.1 9003
src/redis-cli -p 9000 cluster meet 127.0.0.1 9004
src/redis-cli -p 9000 cluster meet 127.0.0.1 9005
```
2.划分master,slave
```
src/redis-cli -c -h 127.0.0.1 -p 9001 cluster nodes
14fca7a90ad3f680c9466c018cb890b5a8717c15 127.0.0.1:9005 master - 0 1468651448427 0 connected
a3fb3e01351afad2aa67affaaef5e2f689c3a007 127.0.0.1:9001 myself,master - 0 0 5 connected
fcb1557f223b6a7e34d1653e3e255f60eedf8d1f 127.0.0.1:9002 master - 0 1468651450831 0 connected
0966d47c01d6725206fa04c0d673072a87cbc76a 127.0.0.1:9003 master - 0 1468651449829 2 connected
5ba760249e5dfc478f5066a98922e0f3268075fd 127.0.0.1:9000 master - 0 1468651448827 1 connected
c0688867266abd0e08781a1acff4d12bf140b017 127.0.0.1:9004 master - 0 1468651451871 3 connected
```
根绝ID指定为哪个node的slave
```
src/redis-cli -c -h 127.0.0.1 -p 9003 cluster replicate 5ba760249e5dfc478f5066a98922e0f3268075fd
src/redis-cli -c -h 127.0.0.1 -p 9004 cluster replicate a3fb3e01351afad2aa67affaaef5e2f689c3a007
src/redis-cli -c -h 127.0.0.1 -p 9005 cluster replicate fcb1557f223b6a7e34d1653e3e255f60eedf8d1f
```
3.分配slot
因为cluster addslots命令不支持区间分配slot，因此只能借助shell实现分配slot
redis.sh如下：
```
#/bin/sh
function addslots()
{
for i in `seq  $2 $3`;do
 src/redis-cli  -p $1 cluster addslots $i  
done
}
case $1 in
    addslots)
    echo "add slots ..."
    addslots $2 $3 $4;
  ;;
  *)
    echo "Usage: inetpanel [addslots]"
  ;;
esac

```
执行shell
```
./redis.sh addslots 9000 0 5460
./redis.sh addslots 9001 5461 10921
./redis.sh addslots 9002 10922 16383
```
至此redis cluster 搭建完毕！
