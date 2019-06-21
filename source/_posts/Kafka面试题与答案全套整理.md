---
title: Kafka面试题与答案全套整理
tags:
  - 技术分享
categories:
  - kafka
toc: false
date: 2019-04-13 19:36:34
---

[toc]
## 声明
本文问题参考[朱小厮的博客](https://mp.weixin.qq.com/s/I-YsRKcjcBv4eOop-jjO0A)。写这个只是为了检验自己最近的学习成果，因为是自己的理解，**如果问题，欢迎指出**。废话少说，上干货。
## 正文
### 1. Kafka的用途有哪些？使用场景如何？
>总结下来就几个字:异步处理、日常系统解耦、削峰、提速、广播

>如果再说具体一点例如:消息,网站活动追踪,监测指标,日志聚合,流处理,事件采集,提交日志等
### 2. Kafka中的ISR、AR又代表什么？ISR的伸缩又指什么
>ISR:In-Sync Replicas 副本同步队列

>AR:Assigned Replicas 所有副本

>ISR是由leader维护，follower从leader同步数据有一些延迟（包括延迟时间replica.lag.time.max.ms和延迟条数replica.lag.max.messages两个维度, 当前最新的版本0.10.x中只支持replica.lag.time.max.ms这个维度），任意一个超过阈值都会把follower剔除出ISR, 存入OSR（Outof-Sync Replicas）列表，新加入的follower也会先存放在OSR中。AR=ISR+OSR。

### 3. Kafka中的HW、LEO、LSO、LW等分别代表什么？
>HW:High Watermark 高水位，取一个partition对应的ISR中最小的LEO作为HW，consumer最多只能消费到HW所在的位置上一条信息。

>LEO:LogEndOffset 当前日志文件中下一条待写信息的offset

>HW/LEO这两个都是指最后一条的下一条的位置而不是指最后一条的位置。

>LSO:Last Stable Offset 对未完成的事务而言，LSO 的值等于事务中第一条消息的位置(firstUnstableOffset)，对已完成的事务而言，它的值同 HW 相同

>LW:Low Watermark 低水位, 代表 AR 集合中最小的 logStartOffset 值

### 4. Kafka中是怎么体现消息顺序性的？
>kafka每个partition中的消息在写入时都是有序的，消费时，每个partition只能被每一个group中的一个消费者消费，保证了消费时也是有序的。
整个topic不保证有序。如果为了保证topic整个有序，那么将partition调整为1.

### 5. Kafka中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？
>拦截器->序列化器->分区器
### 6. Kafka生产者客户端的整体结构是什么样子的？

### 7. Kafka生产者客户端中使用了几个线程来处理？分别是什么？
>2个，主线程和Sender线程。主线程负责创建消息，然后通过分区器、序列化器、拦截器作用之后缓存到累加器RecordAccumulator中。Sender线程负责将RecordAccumulator中消息发送到kafka中.

### 9. Kafka的旧版Scala的消费者客户端的设计有什么缺陷？

### 10. “消费组中的消费者个数如果超过topic的分区，那么就会有消费者消费不到数据”这句话是否正确？如果不正确，那么有没有什么hack的手段？
>不正确，通过自定义分区分配策略，可以将一个consumer指定消费所有partition。
### 11. 消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1?
>offset+1
### 12. 有哪些情形会造成重复消费？
>消费者消费后没有commit offset(程序崩溃/强行kill/消费耗时/自动提交偏移情况下unscrible)
### 13. 那些情景下会造成消息漏消费？
>消费者没有处理完消息 提交offset(自动提交偏移 未处理情况下程序异常结束)
### 14. KafkaConsumer是非线程安全的，那么怎么样实现多线程消费？
>1.在每个线程中新建一个KafkaConsumer

>2.单线程创建KafkaConsumer，多个处理线程处理消息（难点在于是否要考虑消息顺序性，offset的提交方式）

### 15. 简述消费者与消费组之间的关系
>消费者从属与消费组，消费偏移以消费组为单位。每个消费组可以独立消费主题的所有数据，同一消费组内消费者共同消费主题数据，每个分区只能被同一消费组内一个消费者消费。

### 16. 当你使用kafka-topics.sh创建（删除）了一个topic之后，Kafka背后会执行什么逻辑？
>创建:在zk上/brokers/topics/下节点 kafkabroker会监听节点变化创建主题
删除:调用脚本删除topic会在zk上将topic设置待删除标志，kafka后台有定时的线程会扫描所有需要删除的topic进行删除
### 17. topic的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？
>可以
### 18. topic的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？
>不可以
### 19. 创建topic时如何选择合适的分区数？
>根据集群的机器数量和需要的吞吐量来决定适合的分区数
### 20. Kafka目前有那些内部topic，它们都有什么特征？各自的作用又是什么？
>__consumer_offsets 以下划线开头，保存消费组的偏移
### 21. 优先副本是什么？它有什么特殊的作用？
>优先副本 会是默认的leader副本 发生leader变化时重选举会优先选择优先副本作为leader
### 22. Kafka有哪几处地方有分区分配的概念？简述大致的过程及原理
>创建主题时
如果不手动指定分配方式 有两种分配方式

>消费组内分配
### 23. 简述Kafka的日志目录结构
>每个partition一个文件夹，包含四类文件.index .log  .timeindex  leader-epoch-checkpoint
.index .log  .timeindex 三个文件成对出现 前缀为上一个segment的最后一个消息的偏移 log文件中保存了所有的消息 index文件中保存了稀疏的相对偏移的索引 timeindex保存的则是时间索引
leader-epoch-checkpoint中保存了每一任leader开始写入消息时的offset 会定时更新
follower被选为leader时会根据这个确定哪些消息可用

### 24. Kafka中有那些索引文件？
>如上
### 25. 如果我指定了一个offset，Kafka怎么查找到对应的消息？
>1.通过文件名前缀数字x找到该绝对offset 对应消息所在文件

>2.offset-x为在文件中的相对偏移

>3.通过index文件中记录的索引找到最近的消息的位置

>4.从最近位置开始逐条寻找
### 26. 如果我指定了一个timestamp，Kafka怎么查找到对应的消息？
>原理同上 但是时间的因为消息体中不带有时间戳 所以不精确
### 27. 聊一聊你对Kafka的Log Retention的理解
>kafka留存策略包括 删除和压缩两种
删除: 根据时间和大小两个方式进行删除 大小是整个partition日志文件的大小
超过的会从老到新依次删除 时间指日志文件中的最大时间戳而非文件的最后修改时间
压缩: 相同key的value只保存一个 压缩过的是clean 未压缩的dirty  压缩之后的偏移量不连续 未压缩时连续
### 28. 聊一聊你对Kafka的Log Compaction的理解

### 29. 聊一聊你对Kafka底层存储的理解（页缓存、内核层、块层、设备层）

### 30. 聊一聊Kafka的延时操作的原理

### 31. 聊一聊Kafka控制器的作用

### 32. 消费再均衡的原理是什么？（提示：消费者协调器和消费组协调器）

### 33. Kafka中的幂等是怎么实现的
>pid+序号实现，单个producer内幂等
### 33. Kafka中的事务是怎么实现的（这题我去面试6家被问4次，照着答案念也要念十几分钟，面试官简直凑不要脸。实在记不住的话...只要简历上不写精通Kafka一般不会问到，我简历上写的是“熟悉Kafka，了解RabbitMQ....”）

### 34. Kafka中有那些地方需要选举？这些地方的选举策略又有哪些？

### 35. 失效副本是指什么？有那些应对措施？

### 36. 多副本下，各个副本中的HW和LEO的演变过程

### 37. 为什么Kafka不支持读写分离？

### 38. Kafka在可靠性方面做了哪些改进？（HW, LeaderEpoch）

### 39. Kafka中怎么实现死信队列和重试队列？

### 40. Kafka中的延迟队列怎么实现（这题被问的比事务那题还要多！！！听说你会Kafka，那你说说延迟队列怎么实现？）

### 41. Kafka中怎么做消息审计？

### 42. Kafka中怎么做消息轨迹？

### 43. Kafka中有那些配置参数比较有意思？聊一聊你的看法

### 44. Kafka中有那些命名比较有意思？聊一聊你的看法

### 45. Kafka有哪些指标需要着重关注？
>生产者关注MessagesInPerSec、BytesOutPerSec、BytesInPerSec 消费者关注消费延迟Lag
### 46. 怎么计算Lag？(注意read_uncommitted和read_committed状态下的不同)
>参考 [如何监控kafka消费Lag情况](http://trumandu.github.io/2019/04/13/%E5%A6%82%E4%BD%95%E7%9B%91%E6%8E%A7kafka%E6%B6%88%E8%B4%B9Lag%E6%83%85%E5%86%B5/)
### 47. Kafka的那些设计让它有如此高的性能？
>零拷贝，页缓存，顺序写
### 48. Kafka有什么优缺点？

### 49. 还用过什么同质类的其它产品，与Kafka相比有什么优缺点？

### 50. 为什么选择Kafka?
>吞吐量高，大数据消息系统唯一选择。

### 51. 在使用Kafka的过程中遇到过什么困难？怎么解决的？

### 52. 怎么样才能确保Kafka极大程度上的可靠性？

### 53. 聊一聊你对Kafka生态的理解
>confluent全家桶(connect/kafka stream/ksql/center/rest proxy等)，开源监控管理工具kafka-manager,kmanager等
## 参考
1. [Kafka面试题全套整理 | 划重点要考！](https://mp.weixin.qq.com/s/I-YsRKcjcBv4eOop-jjO0A)
2. [Kafka科普系列 | 什么是LSO？](https://blog.csdn.net/u013256816/article/details/88985769)
3. [Kafka面试题](https://www.jianshu.com/p/e4879aed5d43)