---
title: zookeeper运维经验
date: 2017-03-05 20:45:00
tags: 大数据
categories:
- zookeeper
---
# zookeeper运维经验

##  参数配置
在默认zoo.cfg配置中，zookeeper生成的历史镜像,log不会删除，生成的频率也比较快，因此生产环境需要配置以下两个参数

- autopurge.purgeInterval

**解释**：清楚间隔，单位小时，默认值为0，修改次值，表示启用清楚
- autopurge.snapRetainCount

**解释**：保留数量，默认为3，最小值为3



## 引用
1. [zookeeper官网](https://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html#sc_advancedConfiguration)
