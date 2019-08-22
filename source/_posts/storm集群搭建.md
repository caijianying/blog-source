---
title: storm集群搭建
date: 2016-04-15 20:24:41
tags: 大数据
categories:
- storm
---
# storm集群搭建
## 前言
&#160; &#160; &#160; &#160;Storm 是Twitter的一个开源框架。Storm一个分布式的、容错的实时计算系统。Twitter Storm集群表面上类似于Hadoop集群，Hadoop上运行的是MapReduce Jobs，而Storm运行topologies；但是其本身有很大的区别，最主要的区别在于，Hadoop MapReduce Job运行最终会完结，而Storm topologies处理数据进程理论上是永久存活的，除非你将其Kill掉。
Storm集群中包含两类节点：主控节点（Master Node）和工作节点（Work Node）。其分别对应的角色如下：
1.  主控节点（Master Node）上运行一个被称为Nimbus的后台程序，它负责在Storm集群内分发代码，分配任务给工作机器，并且负责监控集群运行状态。Nimbus的作用类似于Hadoop中JobTracker的角色。
2.  每个工作节点（Work Node）上运行一个被称为Supervisor的后台程序。Supervisor负责监听从Nimbus分配给它执行的任务，据此启动或停止执行任务的工作进程。每一个工作进程执行一个Topology的子集；一个运行中的Topology由分布在不同工作节点上的多个工作进程组成。

## 搭建
### 准备软件
```
1.JDK1.8
2.zookeeper-3.4.5-cdh5.6.0
3.Storm0.9.5
```
### JDK安装
详见[略](http:trumandu.github.io/2016/04/15/linux环境jdk安装及配置/)
### Zookeeper集群搭建
1.解压
```
tar xvf zookeeper-3.4.5-cdh5.6.0.tar.gz
```
2.修改配置
添加修改vonf/zoo.cfg
```
cp zoo_sample.cfg zoo.cfg
```
修改zoo.cfg
```
dataDir=/data/truman/zookeeper-3.4.5-cdh5.6.0/data
server.1=lab1:2888:3888
server.2=lab2:2888:3888
server.3=lab3:2888:3888
```
在dataDir下新增myid文件，内容为相对应的server后面的数字
3.分发远程主机
```
scp -r zookeeper-3.4.5-cdh5.6.0 root@lab30:/data/truman/ 
scp -r zookeeper-3.4.5-cdh5.6.0 root@lab31:/data/truman/ 
scp -r zookeeper-3.4.5-cdh5.6.0 root@lab29:/data/truman/ 
```
4.启动和停止
启动命令
```
zkServer.sh start
```
停止
```
zkServer.sh stop
```
### storm集群安装
在nimbus与supervisor节点上重复以下操作
1.修改配置
- storm.zookeeper.servers: 因为Storm所有的信息都是存储在Zookeeper中的，所以要指定Zookeeper服务器的地址
```
storm.zookeeper.servers:
 -  "192.168.0.2"
 -  "192.168.0.3"
 -  "192.168.0.4"
```
- storm.local.dir:
Nimbus和 Supervisor守护进程需要一个目录来存储一些状态信息，例如（ jars, confs, and things like that ）
```
 storm.local.dir: "/data/storm"
```
-  nimbus.host:
worker需要知道那一台机器是master，从而可以下载 topology jars 和confs
```
nimbus.host: "192.168.0.2"
```
-  supervisor.slots.ports
对于每一个supervisor机器，我们可以通过这项来配置运行多少worker在这台机器上。每一个worker使用一个单独的port来接受消息，这个端口同样定义了那些端口是开放使用的。如果你在这里定义了5个端口，就意味着这个supervisor节点上最多可以运行5个worker。如果定义3个端口，则意味着最多可以运行3个worker。在默认情况下(即配置在defaults.yaml中)，会有有四个workers运行在 6700, 6701, 6702, and 6703端口。例如：
```
supervisor.slots.ports:
   - 6700
   - 6701
   - 6702
   - 6703
```
要注意的是：supervisor并不会在启动时就立即启动这四个worker。而是接受到分配的任务时，才会启动，具体启动几个worker也要根据我们Topology在这个supervisor需要几个worker来确定。如果指定Topology只会由一个worker执行，那么supervisor就启动一个worker，并不会启动所有。
- ui端口
```
ui.port: 8998
```
2.启动集群
主节点：执行以下命令
```
nohup $STORM_HOME/bin/storm nimbus &
nohup $STORM_HOME/bin/storm ui &
```
#从节点,执行一下命令
```
nohup $STORM_HOME/bin/storm supervisor &
```
启动成功后，即可在192.168.0.2：8992 storm ui中查看服务
## 参考
1.http://www.tianshouzhi.com/api/tutorials/storm/17
2.http://blog.csdn.net/xeseo/article/details/17678829


