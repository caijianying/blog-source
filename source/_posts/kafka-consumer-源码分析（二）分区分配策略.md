---
title: kafka consumer 源码分析（二）分区分配策略
tags:
  - kafka
  - 专栏
originContent: ''
categories:
  - kafka
toc: false
date: 2019-06-27 23:04:18
---

# kafka consumer 源码分析（二）分区分配策略

## 开篇

在开始这篇之前，先抛出问题，这章主要通过研究consumer源码解决如下问题：

分区分配策略是什么样的？

## 正文

### 分区分配策略

这里的分区分配策略仅仅只讨论consumer,关于producer端的分区不在本次探讨范围内。

consumer端分区分配策略官方有三种：`RangeAssignor`（默认值）,`RoundRobinAssignor`,`StickyAssignor`

通常是通过`partition.assignment.strategy`配置生效。

#### RangeAssignor

```java
org.apache.kafka.clients.consumer.RangeAssignor#assign

Collections.sort(consumersForTopic);//consumer按字典排序
int numPartitionsPerConsumer = numPartitionsForTopic / consumersForTopic.size();
int consumersWithExtraPartition = numPartitionsForTopic % consumersForTopic.size();
List<TopicPartition> partitions = AbstractPartitionAssignor.partitions(topic,numPartitionsForTopic);
int i = 0;
for(int n = consumersForTopic.size(); i < n; ++i) {
     int start = numPartitionsPerConsumer * i + Math.min(i, consumersWithExtraPartition);
     int length = numPartitionsPerConsumer + (i + 1 > consumersWithExtraPartition ? 0 : 1);
     ((List)assignment.get(consumersForTopic.get(i))).addAll(partitions.subList(start, start + length));
}
```

源码太多，这里只贴出核心代码，需要的按照以上目录去找。

**例如**：

有一个topic:**truman** partition:**10** consumer:**consumer0,consumer1,consumer2**。那么numPartitionsPerConsumer=10/3=3，consumersWithExtraPartition=10%3=1。分配逻辑为：当i=0,按字典顺序，先分配consumer0，start=0，length=4，即consumer1分配tp0,tp1,tp2,tp3。同理最终方案为：

```
consumer0->tp0,tp1,tp2,tp3
consumer1->tp4,tp5,tp6
consumer2->tp7,tp8,tp9
```



从源码我们能得出如何分配的，还能得出一个结论，consumer分配partition和consumerId有关系，字典顺序靠前的，有可能会多分配partition。

为什么这么说？上面的源码其实没有给出，但能看出来consumersForTopic集合是按字典排序的，那么只用知道consumersForTopic是什么样的集合就能知道了，同样在该源码中可以看到`put(res, topic, consumerId);`,说明该集合是consumerId集合。因此可以得出该结论！

**一句话总结该分配策略，就是尽可能的均分**

### RoundRobinAssignor

```java
org.apache.kafka.clients.consumer.RoundRobinAssignor
//assigner存放了所有的consumer
CircularIterator<String> assigner = new CircularIterator(Utils.sorted(subscriptions.keySet()));
//所有的consumer订阅的所有的TopicPartition的List
Iterator var9 = this.allPartitionsSorted(partitionsPerTopic, subscriptions).iterator();

while(var9.hasNext()) {
 TopicPartition partition = (TopicPartition)var9.next();
 String topic = partition.topic();
 // 如果当前这个assigner(consumer)没有订阅这个topic，直接跳过
 while(!((Subscription)subscriptions.get(assigner.peek())).topics().contains(topic)) {
 assigner.next();
}
//跳出循环，表示终于找到了订阅过这个TopicPartition对应的topic的assigner
//将这个partition分派给对应的assigner
((List)assignment.get(assigner.next())).add(partition);
}
```

CircularIterator是一个封装类，让一个有限大小的list变成一个RoundRobin方式的无限遍历

RoundRobinAssignor与RangeAssignor最大的区别，是进行分区分配的时候不再逐个topic进行，即不是为某个topic完成了分区分派以后，再进行下一个topic的分区分派，而是首先将这个group中的所有consumer订阅的所有的topic-partition按顺序展开，然后，依次对于每一个topic-partition，在consumer进行round robin，为这个topic-partition选择一个consumer。

**例如**：

有两个topic :t0和t1，每个topic都有3个分区，分区编号从0开始，因此展开以后得到的TopicPartition的List是：[t0p0,t0p1,t0p2,t1p0,t1p1,t1p2]

我们有两个consumer C0,C1,他们都同时订阅了这两个topic。

然后，在这个展开的TopicParititon的List开始进行分派：

t0p0 分配给C0
t0p1 分配给C1
t0p2 分配给C0
t1p0 分配给C1
t1p1 分配给C0
t1p1 分配给C1
从以上分配流程，我们可以很清楚地看到RoundRobinAssignor的两个基本特征：

1. 对所有topic的topic partition求并集
2. 基于consumer进行RoundRobin轮询

最终分配方案：

```
C0->[t0p0, t0p2, t1p1]
C1->[t0p1, t1p0, t1p2]
```

**RoundRobinAssignor与RangeAssignor的重大区别，就是RoundRobinAssignor是在Group中所有consumer所关注的全体topic中进行分派，而RangeAssignor则是依次对每个topic进行分派。**

那么什么时候选择RoundRobinAssignor，我们再看如下案例：

假如现在有两个Topic:Ta和Tb ，每个topic都有两个partition，同时，有两个消费者Ca和Cb与Cc，这三个消费者全部订阅了这两个主题，那么，通过两种不同的分区分派算法得到的结果将是：

RangeAssignor:

先对Ta进行处理：会将TaP0分派给Ca，将TaP1分派给Cb

在对Tb进行处理：会将TbP0分派给Ca，将TbP1分派给Cb

最终结果是：

```
Ca:[TaP0,TbP0]
Cb:[TaP1,TbP1]
Cc:[]
```

RoundRobinAssignor：

将所有TopicPartition展开，变成:[TaP0,TaP1,TbP0,TbP1]

将TaP0分派给Ca，将TaP1分派给Cb，将TbP0分派给Cc，将TbP1分派给Ca

最终分派结果是：

```
Ca:[TaP0,TbP1]
Cb:[TaP1]
Cc:[TbP0]
```

### StickyAssignor

这个分配器源码较多，实现也要优于RoundRobinAssignor与RangeAssignor

它的主要特点是分区的分配尽可能的与上次分配的保持相同。当两者发生冲突时，均衡优先于上一次分配结果。鉴于这两个目标。



## 参考

1. [Kafka为Consumer分派分区：RangeAssignor和RoundRobinAssignor](https://blog.csdn.net/zhanyuanlin/article/details/76021614)
2. [Kafka分区分配策略（2）——RoundRobinAssignor和StickyAssignor](https://blog.csdn.net/u013256816/article/details/81123625)