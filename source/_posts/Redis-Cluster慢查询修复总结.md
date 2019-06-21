---
title: Redis Cluster慢查询修复总结
date: 2016-08-28 19:27:16
tags: nosql
categories:
- redis
---
## RedisCluster集群定位
### 故障排除

如果在产线中Redis 操作变慢，可按以下操作排除故障

- slowlog查看引发延迟的慢命令

获取10条慢操作日志
```
slowlog get 10
```
结果为查询ID、发生时间、运行时长和原命令。默认10毫秒，默认只保留最后的128条。单线程的模型下，一个请求占掉10毫秒是件大事情，注意设置和显示的单位为**微秒**，注意这个时间是不包含网络延迟的。

- 监控客户端的连接

在Redis-cli工具中输入info clients可以查看到当前实例的所有客户端连接信息,过多的连接数也会影响redis性能
```
info clients
```

- 机器固有延迟

如果使用的是虚拟机的话，会存在一个固有延迟.可以使用如下命令，在redis server中执行：
```
./redis-cli --intrinsic-latency 100
```
100位一秒内测试数量

- 计算延迟

如果你正在经历redis 延迟问题，可能需要衡量延迟到底是多大，用以下命令即可实现
```
redis-cli --latency -h `host` -p `port`
```

- AOF和磁盘IO引起延迟

查看redis中fdatasync(2)，追踪进程
```
sudo strace -p $(pidof redis-server) -T -e trace=fdatasync,write
```

- 过期key引起延迟

当数据库有超过1/4key在同一时间过期，会引起redis 阻塞


### 产线调整

- aof配置调整

经过查看redis log,我们发现aof延迟比较厉害，猜测写aof影响了redis 性能。但为了保证数据安全，又无法关闭aof文件。经过查看官方文档，未更改aof保存策略维持fsync every second，更改
```
no-appendfsync-on-rewrite yes
```
这个配置是指在重写aof时，不持久化数据到aof中。

- rdb配置调整

线上机器IO操作特别频繁，用命令
```
iostat -d -k 2
```
查看当前IO情况。经过查看发现redis rdb temp文件变更比较频繁。将配置改为
```
save 600 30000
```
线上机器IO才降下来。
## Client使用定位
- pipeline操作串行改并行

经过测试，高并发情况下，key批量pipeline处理已经成为性能瓶颈。经过分析将这块的处理更改成并行处理，并行处理的线程数为当前master的数量，这样会将Client的处理能力提升n倍，经过测试，效果明显。
- 集群节点信息单独线程维护

之前Client为了实时获取集群node信息，在每次操作都会主动获取集群信息。这里操作不用这么频繁，如果并发特别大的话，也会消耗大量的时间，经过分析后将此处做更改。有两种修改方法(1.)单独新建线程，专门维护集群信息（2.）更改成处理失败情况下，再去更新集群信息。目前采用第二种
- 增加slow log记录

系统如果不存在对操作时间log统计，那么就会出现不容易定位是哪一块出的问题，但也是一把双刃剑，增加slow  log 统计会浪费时间，增加系统负担。基于以上综合考虑：特将slow log架构设计成通过rest API动态改变统计维度，统计分为set和get两种(set数据理论上会比get数据慢)。

**问题1**

更改成并行处理后，出现许多莫名其妙的问题，错误如下：
```
redis.clients.jedis.exceptions.JedisConnectionException: Unexpected end of stream.
```
**解决**
分析log,发现有同时多线程处理同一个key情况，更改key处理顺序，避免多线程同时操作一个key，此问题再未复现。

**问题2**

更改成并行处理后，出现开始一部分操作报null错误

**解决**

分析log,发现多线程同时调用一个获取集群信息方法，该方法避免多次调用，已经加lock,这会导致多数线程得不到集群信息，因此会出现null,将该方法提到初始化时即调用，彻底避免多线程调用。

## 参考
以下链接为我收藏的网址，对追踪Redis latency有很大帮助
1. http://redis.io/topics/latency
2. http://www.tuicool.com/articles/2Q7nauM
3. http://www.jianshu.com/p/ee2aa7fe341b
4. http://www.cnblogs.com/mushroom/p/4738170.html

