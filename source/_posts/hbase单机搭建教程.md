---
title: hbase单机搭建教程
date: 2016-03-19 19:58:41
tags: 大数据
categories:
- hbase
---
# 搭建hbase单机版
## 目的
&emsp;本教程为了搭建一个hbas单机版本，搭建好可以便于练习使用hbase,选用的版本不是最新的，但搭建原理都是一样的。单机版数据存储在本地文件上，使用自带的zookeeper。
## 准备
1.[下载](http:http://www.gtlib.gatech.edu/pub/apache/hbase/hbase-0.94.27/)hbase-0.94.27 
## 安装与部署
- 安装
<pre><code>
tar -xvf hbase-0.94.27.tar.gz 
</code></pre>
<!--more-->
- 配置
1.**编辑conf/hbase-env.sh**
<pre><code>
export JAVA_HOME=/opt/jdk1.8.0_45
export HBASE_MANAGES_ZK=true
</code></pre>
2.**编辑conf/hbase-site.xml**
<pre><code>
<configuration>
      <property>
          <name>hbase.rootdir</name>
          <value>file:///data/truman/hbasedata</value>
       </property>
       <property>
           <name>hbase.tmp.dir</name>
           <value>/data/truman/hbasedata/log</value>
       </property>
       <property>
           <name>hbase.zookeeper.property.dataDir</name>
           <value>/data/truman/hbasedata/logs/zookeeper</value>
      </property>
</configuration>
</code></pre>
## 启动

1.启动

<pre><code>
cd hbase-0.94.27
bin/start-hbase.sh
</code></pre>

2.连接测试

测试成功结果如下：
<pre><code>
[root@LAB hbase-0.94.3]# bin/hbase shell
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 0.94.3, r1408904, Wed Nov 14 19:55:11 UTC 2012
hbase(main):001:0> list
TABLE
emp
t1
2 row(s) in 1.0670 seconds
hbase(main):002:0>
</code></pre>

