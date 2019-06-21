---
title: RedisCluster构建批量操作探讨
date: 2016-05-09 09:50:03
tags: nosql
categories:
- redis
---
# RedisCluster构建批量操作探讨
## 前言 ##
众所周知，jedis仅支持redis standalone mset,mget等批量操作，在最新的redis cluster中是不支持的，这个和redis cluster的设计有关，将不同的实例划分不同的槽。不同的key会落到不同的槽上，所在的实例也就不同，这就对jedis的批量操作提出问题，即无法同时支持多个实例上的批量操作。（理解可能不是很深入，希望有人看到可以指教一下）
### 分布式储存产品存储方式
在分布式存储产品中，哈希存储与顺序存储是两种重要的数据存储和分布方式，这两种方式不同也直接决定了批量获取数据的不同，所以这里需要对这两种数据的分布式方式进行简要说明：
   1. hash分布：
   hash分布应用于大部分key-value系统中，例如memcache, redis-cluster, twemproxy，即使像MySQL在分库分表时候，也经常会用user%100这样的方式。
   hash分布的主要作用是将key均匀的分布到各个机器，所以它的一个特点就是数据分散度较高，实现方式通常是hash(key)得到的整数再和分布式节点的某台机器做映射，以redis-cluster为例子：
    ![](http://dl2.iteye.com/upload/attachment/0113/6187/1b8483c0-502e-3a3b-93ca-4bcf12b01e89.jpg)
   问题：和业务没什么关系，不支持范围查询。
   2. 顺序分布
 ![](http://dl2.iteye.com/upload/attachment/0113/6193/def09084-469c-3513-9dab-8e789a2c20c2.jpg) 
   3. 两种分布方式的比较：
   
| 分布方式 | 特点 | 典型产品 |
| -------- | ---- | -------- |
| 哈希分布 | 1. 数据分散度高</br>2.键值分布与业务无关</br>3.无法顺序访问</br>4.支持批量操作 | 一致性哈希memcache </br>redisCluster其他缓存产品 |
| 顺序分布 | 1.数据分散度易倾斜</br>2.键值分布与业务相关</br>3.可以顺序访问</br>4.支持批量操作 | BigTable </br>Hbase |

## 分布式缓存/存储四种Mget解决方案
### 1. IO的优化思路：
  * 命令本身的效率：例如sql优化，命令优化
  * 网络次数：减少通信次数
  * 降低接入成本:长连/连接池,NIO等。
  * IO访问合并:O(n)到O(1)过程:批量接口(mget),

### 2. 如果只考虑减少网络次数的话，mget会有如下模型：
![](http://dl2.iteye.com/upload/attachment/0113/6195/8dc22bf0-c55a-3a29-8285-4a11ef0c67be.jpg)

### 3. 四种解决方案：
#### (1).串行mget
将Mget操作(n个key)拆分为逐次执行N次get操作, 很明显这种操作时间复杂度较高，它的操作时间=n次网络时间+n次命令时间，网络次数是n，很显然这种方案不是最优的，但是足够简单。
 ![](http://dl2.iteye.com/upload/attachment/0113/5649/734b8416-0dde-3ea5-bd85-6b19c058d8ee.png)
#### (2). 串行IO
将Mget操作(n个key)，利用已知的hash函数算出key对应的节点，这样就可以得到一个这样的关系：Map<node, somekeys>，也就是每个节点对应的一些keys 。它的操作时间=node次网络时间+n次命令时间，网络次数是node的个数，很明显这种方案比第一种要好很多，但是如果节点数足够多，还是有一定的性能问题。
 ![](http://dl2.iteye.com/upload/attachment/0113/5599/a6e24459-5bca-3c42-b555-97f3c7c2d4f7.png)
#### (3). 并行IO
此方案是将方案（2）中的最后一步，改为多线程执行，网络次数虽然还是nodes.size()，但网络时间变为o(1)，但是这种方案会增加编程的复杂度。它的操作时间=1次网络时间+n次命令时间
 ![](http://dl2.iteye.com/upload/attachment/0113/5653/668355e5-34f7-30a2-aee3-b4eb8b8dae68.png)
#### (4).hash-tag实现
第二节提到过，由于hash函数会造成key随机分配到各个节点，那么有没有一种方法能够强制一些key到指定节点到指定的节点呢? redis提供了这样的功能，叫做hash-tag。什么意思呢？假如我们现在使用的是redis-cluster（10个redis节点组成），我们现在有1000个k-v，那么按照hash函数(crc16)规则，这1000个key会被打散到10个节点上，那么时间复杂度还是上述(1)~(3)
 ![](http://dl2.iteye.com/upload/attachment/0113/5655/0cc9ab10-39ec-3114-a82a-68cb7d5075e4.png)
那么我们能不能像使用单机redis一样，一次IO将所有的key取出来呢？hash-tag提供了这样的功能，如果将上述的key改为如下，也就是用大括号括起来相同的内容，那么这些key就会到指定的一个节点上。
例如：
 ![](http://dl2.iteye.com/upload/attachment/0113/5657/8b42b6fb-91d0-367b-b72d-fd01f81c78d4.png)
例如下图：它的操作时间=1次网络时间+n次命令时间
![](http://dl2.iteye.com/upload/attachment/0113/5603/b17e3697-9d98-39ae-9500-a4365a3b2c69.png)
### 4. 四种批量操作解决方案对比：
|方案|优点|缺点|网络IO|
|----|----|----|------|
|串行mget|1.编程简单</br>2.少量keys，性能满足要求|大量keys请求延迟严重|o(keys)|
|串行IO|1.编程简单</br>2.少量节点，性能满足要求|大量node延迟严重|o(nodes)|
|并行IO|1.利用并行特性</br>2.延迟取决于最慢的节点|1.编程复杂</br>2.超时定位较难|o(max_slow(node))|
|hash tags|性能最高|1.tag-key业务维护成本较高</br>2.tag分布容易出现数据倾斜|o(1)|


## 总结 
目前我们项目中采用第一种方式，使用多线程解决批量问题，减少带宽时延，提高效率，这种做法就如上面所说简单便捷（我们目前批量操作类型比较多），有效。但问题比较明显。批量操作数量不大即可满足。最近研究cachecloud发现搜狐他们采用第二点，先将key获取槽点，然后分node pipeline操作。这种做法相对比第一种做法较优。推荐第二种方法，后期对性能有要求的话，考虑修改成第二种方式。
附加cachecloud-client代码中mget追踪路径
类PipelineCluster======>mget======>类pipelineCommand=====>run=====>getPoolKeyMap====>getResult
## 参考
[1]http://carlosfu.iteye.com/blog/2263813
[2]https://github.com/sohutv/cachecloud/wiki/5.%E8%AE%BE%E8%AE%A1%E6%96%87%E6%A1%A3

