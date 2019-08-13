---
title: Redis4.0 新特性尝鲜
date: 2017-07-19 22:17:00
tags: nosql
categories:
- redis
---
# Redis4.0 新特性尝鲜

## 1.目录结构

1. [目录](#1-目录结构)
2. [前言](#2-前言)
3. [环境搭建](#3-环境搭建)
   1. [下载](#31-下载)
   2. [安装](#32-编译安装)
   3. [集群搭建](#33-新建集群)
4. [特性尝鲜](#4-特性尝鲜)
   1. [注意事项](#41-注意事项)
   2. [升级内容](#42-升级内容)
   3. [模块系统](#43-模块系统)
   4. [PSYNC 2.0](#44-PSYNC)
   5. [缓存驱逐策略优化](#45-缓存驱逐策略优化)
   6. [非阻塞 DEL 、 FLUSHDB 和 FLUSHALL](#46-)
   7. [交换数据库](#47-交换数据库)
   8. [混合RDBAOF持久化格式](#48-)
   9. [内存命令](#49-内存命令)
   10. [兼容 NAT 和 Docker](#410-兼容NAT和Docker)
5. [引用](#5-引用)

## 2. 前言

**2017-07-14 redis 4.0  Stable  version release**,新增了许多新功能，此次专门抽出时间，探索一些功能，把握Redis未来的发展方向，同时积累经验，为未来升级Redis打下坚实基础。学习新的东西，如果不将它记录下来，过上两周，基本上就忘记做过什么了，对知识的掌握不利，以后尽量所有的学习都能产生文字记录，便于自己总结学习各种技术，也能给新人带去一点便利。

## 3. 环境搭建

安装方式，较之前没有多大变化，还是写一下，便于新手学习。

### 3.1. 下载
```
$ wget http://download.redis.io/releases/redis-4.0.0.tar.gz
$ tar xzf redis-4.0.0.tar.gz
```

### 3.2. 编译安装
```
$ cd redis-4.0.0
$ make
```
如果需要将redis-cli,redis-server等相关命令安装到/bin目录下，全局使用的话，可以使用如下命令：
```
$ make install
```
或者利用软连接实现：
```
$ make
$ ln -s  redis-4.0.0/src/redis-server  /bin/redis-server
$ ln -s  redis-4.0.0/src/redis-cli  /bin/redis-cli
```
在编译中可能会遇到如下问题：
```
 zmalloc.h:50:31: 错误：jemalloc/jemalloc.h：没有那个文件或目录
```
出现这个问题是libc 并不是默认的 分配器， 默认的是 jemalloc, 因为 jemalloc 被证明 有更少的 fragmentation problems 比libc，关于更详细信息可以查看redis中REAME,md，其中有详细介绍，此处不再细说。

解决办法：
```
make MALLOC=libc
```

### 3.3. 新建集群

按照官方指导，搭建3m+3s集群，端口号7000-7005

以下为创建脚本：

```
#!/bin/bash
 nodes=(7000 7001 7002 7003 7004 7005)
 HOME_DIR=`pwd`
function create(){
 echo "create"
 if [ -d redis_cluster ];then
 rm -rf  redis_cluster
 fi

 mkdir  redis_cluster
 cd redis_cluster

 mkdir `echo ${nodes[*]}`

 for var in ${nodes[*]}
 do
  cd $var
  touch $var-redis.conf
  echo "port" $var >> $var-redis.conf
  echo "cluster-enabled yes" >> $var-redis.conf
  echo "cluster-config-file nodes.conf" >> $var-redis.conf
  echo "cluster-node-timeout 5000" >> $var-redis.conf
  echo "appendonly yes" >> $var-redis.conf
  echo "daemonize yes" >> $var-redis.conf
  echo "pidfile redis.pid" >> $var-redis.conf
  cd ../
 done
 run;
 src/redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005

}

function run(){
echo run
 for var in ${nodes[*]}
 do
   cd $HOME_DIR/redis_cluster/$var/
   redis-server $var-redis.conf
   cd $HOME_DIR/
   echo Start redis ports ${var} finish.
 done
}
function stop(){
 cd $HOME_DIR/redis_cluster/
 DIRS=`ls -l | grep "^d" | awk '{print $NF}'`
  for d in $DIRS
  do
    PID=`cat $d/redis.pid`
    if [ -n "$PID" ];then
       kill -9 $PID
       rm -f $d/redis.pid
    fi
  done
  cd  $HOME_DIR/
  echo Stop redis ports ${DIRS} finish.
}


case $1 in
     create)
      echo " create redis cluster 锛?000,7001,7002,7003,7004.7005,7006,7007"
      create
      ;;
     run)
      run
      ;;
     stop)
      stop
     ;;
     *)
      echo "Usage:[create|run|stop]"
      ;;
esac
```

执行

```
sh createCluster.sh create
```
一路回车即可，注意构建集群需要使用redis-trib.rb，需要额外执行

```
gem install redis
```
结果如下：

```
[bigdata@trumanlab1 redis-4.0.0]$ redis-cli -c -h 127.0.0.1  -p 7000
127.0.0.1:7000> cluster nodes
63e698210378f4fda23c4070bea3b46f93952811 127.0.0.1:7000@17000 myself,master - 0 1500391716000 1 connected 0-5460
4091630217b64ece9a6fa55c26687a10c1f9a8b5 127.0.0.1:7001@17001 master - 0 1500391717501 2 connected 5461-10922
0b23853c3aa4f5b942a6239bc7bca71df36e121c 127.0.0.1:7005@17005 slave 12d05fb7c20bf001759f80cf3f65ad9df4b01af5 0 1500391716498 6 connected
12d05fb7c20bf001759f80cf3f65ad9df4b01af5 127.0.0.1:7002@17002 master - 0 1500391717000 3 connected 10923-16383
2e559b462362a9563f98b398af3984c05e25ef19 127.0.0.1:7003@17003 slave 63e698210378f4fda23c4070bea3b46f93952811 0 1500391716000 4 connected
c4f8e0a95579c772c7506c195b3c7984c73312b4 127.0.0.1:7004@17004 slave 4091630217b64ece9a6fa55c26687a10c1f9a8b5 0 1500391716000 5 connected
127.0.0.1:7000> set a a
-> Redirected to slot [15495] located at 127.0.0.1:7002
OK
127.0.0.1:7002> get a
"a"
127.0.0.1:7002>
```

## 4. 特性尝鲜

### 4.1 注意事项

> IMPORTANT: Redis Cluster users, please note that, as specified in the list
of incompatibilities, Redis 4.0 cluster bus protocol is not compatible with
Redis 3.2, so in order to upgrade, a mass reboot of the instances is needed
and rolling upgrades are not possible. This change was needed in order to
add compatibility for Containers/NAT, where the bus port at a fixed offset
was not an acceptable design, so we had to change many things, resulting
in the incompatible protocol.

4.0版本与3.2版本传输协议不兼容，更改该协议的目的是为了实现Containers/NAT
为了升级，需要重启大量节点，无法做到滚动升级。

### 4.2 升级内容
>* Different replication fixes to PSYNC2, the new 4.0 replication engine.
>* Modules thread safe contexts were introduced. They are an >experimental API right now, but the API is considered to be stable and usable when needed.
>* SLOWLOG now logs the offending client name and address. Note that this is a backward compatibility breakage in case old code assumes that the slowlog entry is composed of exactly three entries.
>* The modules native data types RDB format changed.
>* The AOF check utility is now able to deal with RDB preambles.
>* GEORADIUS_RO and GEORADIUSBYMEMBER_RO variants, not supporting the STORE option, were added in order to allow read-only scaling of such queries.
>* HSET is now variadic, and HMSET is considered deprecated (but will be supported for years to come). Please use HSET in new code.
>* GEORADIUS huge radius (>= ~6000 km) corner cases fixed, certain elements near the edges were not returned.
>* DEBUG DIGEST modules API added.
>* HyperLogLog commands no longer crash on certain input (non HLL) strings.
>* Fixed SLAVEOF inside MULTI/EXEC blocks.
>* Many other minor bug fixes and improvements.

### 4.3 模块系统
Redis 4.0 发生的最大变化就是加入了模块系统， 这个系统可以让用户通过自己编写的代码来扩展和实现 Redis 本身并不具备的功能， 具体使用方法可以参考 antirez 的博文《Redis Loadable Module System》： http://antirez.com/news/106
因为模块系统是通过高层次 API 实现的， 它与 Redis 内核本身完全分离、互不干扰， 所以用户可以在有需要的情况下才启用这个功能， 以下是 redis.conf 中记载的模块载入方法：
```
################################## MODULES #####################################

# Load modules at startup. If the server is not able to load modules
# it will abort. It is possible to use multiple loadmodule directives.
#
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so
```
目前已经有人使用这个功能开发了各种各样的模块， 比如 Redis Labs 开发的一些模块就可以在 http://redismodules.com 看到， 此外 antirez 自己也使用这个功能开发了一个神经网络模块： https://github.com/antirez/neural-redis
模块功能使得用户可以将 Redis 用作基础设施， 并在上面构建更多功能， 这给 Redis 带来了无数新的可能性。

### 4.4 PSYNC 2.0

新版本的 PSYNC 命令解决了旧版本的 Redis 在复制时的一些不够优化的地方：
- 在旧版本 Redis 中， 如果一个从服务器在 FAILOVER 之后成为了新的主节点， 那么其他从节点在复制这个新主的时候就必须进行全量复制。 在 Redis 4.0 中， 新主和从服务器在处理这种情况时， 将在条件允许的情况下使用部分复制。
- 在旧版本 Redis 中， 一个从服务器如果重启了， 那么它就必须与主服务器重新进行全量复制， 在 Redis 4.0 中， 只要条件允许， 主从在处理这种情况时将使用部分复制。

### 4.5 缓存驱逐策略优化

新添加了 Last Frequently Used 缓存驱逐策略， 具体信息见 antirez 的博文《Random notes on improving the Redis LRU algorithm》： http://antirez.com/news/109
另外 Redis 4.0 还对已有的缓存驱逐策略进行了优化， 使得它们能够更健壮、高效、快速和精确。

### 4.6 非阻塞 DEL 、 FLUSHDB 和 FLUSHALL

在 Redis 4.0 之前， 用户在使用 DEL 命令删除体积较大的键， 又或者在使用 FLUSHDB 和 FLUSHALL 删除包含大量键的数据库时， 都可能会造成服务器阻塞。
为了解决以上问题， Redis 4.0 新添加了 UNLINK 命令， 这个命令是 DEL 命令的异步版本， 它可以将删除指定键的操作放在后台线程里面执行， 从而尽可能地避免服务器阻塞：
```

127.0.0.1:7002> set a test
OK
127.0.0.1:7002> get a
"test"
127.0.0.1:7002> unlink a
(integer) 1
127.0.0.1:7002> get a
(nil)

```
因为一些历史原因， 执行同步删除操作的 DEL 命令将会继续保留。
此外， Redis 4.0 中的 FLUSHDB 和 FLUSHALL 这两个命令都新添加了 ASYNC 选项， 带有这个选项的数据库删除操作将在后台线程进行：
```
redis> FLUSHDB ASYNC
OK

redis> FLUSHALL ASYNC
OK
```

### 4.7 交换数据库
Redis 4.0 对数据库命令的另外一个修改是新增了 SWAPDB 命令， 这个命令可以对指定的两个数据库进行互换： 比如说， 通过执行命令 SWAPDB 0 1 ， 我们可以将原来的数据库 0 变成数据库 1 ， 而原来的数据库 1 则变成数据库 0 。这种场景是在非集群模式下使用的，在集群模式是不支持切换数据库的。默认所有的节点使用index 0。
以下是一个使用 SWAPDB 的例子：
```
redis> SET your_name "aibibang"  -- 在数据库 0 中设置一个键
OK

redis> GET your_name
"aibibang"

redis> SWAPDB 0 1  -- 互换数据库 0 和数据库 1
OK

redis> GET your_name  -- 现在的数据库 0 已经没有之前设置的键了
(nil)

redis> SELECT 1  -- 切换到数据库 1
OK

redis[1]> GET your_name  -- 之前在数据库 0 设置的键现在可以在数据库 1 找到
"aibibang"                 -- 证明两个数据库已经互换

```

### 4.8 混合 RDB-AOF 持久化格式

Redis 4.0 新增了 RDB-AOF 混合持久化格式， 这是一个可选的功能， 在开启了这个功能之后， AOF 重写产生的文件将同时包含 RDB 格式的内容和 AOF 格式的内容， 其中 RDB 格式的内容用于记录已有的数据， 而 AOF 格式的内存则用于记录最近发生了变化的数据， 这样 Redis 就可以同时兼有 RDB 持久化和 AOF 持久化的优点 —— 既能够快速地生成重写文件， 也能够在出现问题时， 快速地载入数据。
这个功能可以通过 aof-use-rdb-preamble 选项进行开启， redis.conf 文件中记录了这个选项的使用方法：
```
 When rewriting the AOF file, Redis is able to use an RDB preamble in the
# AOF file for faster rewrites and recoveries. When this option is turned
# on the rewritten AOF file is composed of two different stanzas:
#
#   [RDB file][AOF tail]
#
# When loading Redis recognizes that the AOF file starts with the "REDIS"
# string and loads the prefixed RDB file, and continues loading the AOF
# tail.
#
# This is currently turned off by default in order to avoid the surprise
# of a format change, but will at some point be used as the default.
aof-use-rdb-preamble no

```

### 4.9 内存命令
新添加了一个 MEMORY 命令， 这个命令可以用于视察内存使用情况， 并进行相应的内存管理操作：
```
127.0.0.1:7002> memory help
1) "MEMORY USAGE <key> [SAMPLES <count>] - Estimate memory usage of key"
2) "MEMORY STATS                         - Show memory usage details"
3) "MEMORY PURGE                         - Ask the allocator to release memory"
4) "MEMORY MALLOC-STATS                  - Show allocator internal stats"

```
其中， 使用 MEMORY USAGE 子命令可以估算储存给定键所需的内存：
```

127.0.0.1:7002> set truman aibibang
-> Redirected to slot [2113] located at 127.0.0.1:7000
OK
127.0.0.1:7000> memory usage truman
(integer) 59
```
使用 MEMORY STATS 子命令可以查看 Redis 当前的内存使用情况：
```
127.0.0.1:7000> memory stats
 1) "peak.allocated"
 2) (integer) 2228415
 3) "total.allocated"
 4) (integer) 2228474
 5) "startup.allocated"
 6) (integer) 1054396
 7) "replication.backlog"
 8) (integer) 1048584
 9) "clients.slaves"
10) (integer) 16858
11) "clients.normal"
12) (integer) 49630
13) "aof.buffer"
14) (integer) 0
15) "db.0"
16) 1) "overhead.hashtable.main"
    2) (integer) 112
    3) "overhead.hashtable.expires"
    4) (integer) 0
17) "overhead.total"
18) (integer) 2169580
19) "keys.count"
20) (integer) 2
21) "keys.bytes-per-key"
22) (integer) 587039
23) "dataset.bytes"
24) (integer) 58894
25) "dataset.percentage"
26) "5.0161914825439453"
27) "peak.percentage"
28) "100.00264739990234"
29) "fragmentation"
30) "1.2773568630218506"
```
使用 MEMORY PURGE 子命令可以要求分配器释放更多内存：
```
redis> MEMORY PURGE
OK
```
使用 MEMORY MALLOC-STATS 子命令可以展示分配器内部状态：
```
redis> MEMORY MALLOC-STATS
Stats not supported for the current allocator
```

### 4.10 兼容NAT和Docker
Redis 4.0 将兼容 NAT 和 Docker ， 具体的使用方法在 redis.conf 中有记载：

```
########################## CLUSTER DOCKER/NAT support  ########################
# In certain deployments, Redis Cluster nodes address discovery fails, because
# addresses are NAT-ted or because ports are forwarded (the typical case is
# Docker and other containers).
#
# In order to make Redis Cluster working in such environments, a static
# configuration where each node known its public address is needed. The
# following two options are used for this scope, and are:
#
# * cluster-announce-ip
# * cluster-announce-port
# * cluster-announce-bus-port
#
# Each instruct the node about its address, client port, and cluster message
# bus port. The information is then published in the header of the bus packets
# so that other nodes will be able to correctly map the address of the node
# publishing the information.
#
# If the above options are not used, the normal Redis Cluster auto-detection
# will be used instead.
#
# Note that when remapped, the bus port may not be at the fixed offset of
# clients port + 10000, so you can specify any port and bus-port depending
# on how they get remapped. If the bus-port is not set, a fixed offset of
# 10000 will be used as usually.
#
# Example:
#
# cluster-announce-ip 10.1.1.5
# cluster-announce-port 6379
# cluster-announce-bus-port 6380
```
## 5. 引用

1. [Redis 4.0新功能介绍](http://blog.csdn.net/yin767833376/article/details/53518272)
2. [Redis 4.0 release note](https://github.com/antirez/redis/blob/05b81d2b02578d432329c87c93f975e582d14c0e/00-RELEASENOTES)
