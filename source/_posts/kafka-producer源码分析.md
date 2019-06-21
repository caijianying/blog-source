---
title: kafka producer源码分析
tags:
  - kafka
  - 专栏
categories:
  - kafka
toc: false
date: 2019-06-14 21:47:42
---

 Kafka producer源码分析

## 前言

在开始文章之前，需要解释是一下为什么要研究producer源码。

### 为什么要研究producer源码

通常producer使用都很简单，初始化一个`KafkaProducer`实例，然后调用`send`方法就好，但是我们有了解后面是如何发送到kafka集群的吗？其实我们不知道，其次，到底客户端有几个线程？我们不知道。还有producer还能做什么？我们同样不知道。本篇文章就是想回答一下上面提出的几个问题，能力有限，如有错误，欢迎指出！

## 架构

在介绍客户端架构之前，先回答一个问题

producer到底存在几个线程？**2个**  **Main thread** 和**sender**,其中sender线程负责发送消息，main 线程负责interceptor、序列化、分区等其他操作。

![](http://blogstatic.aibibang.com/Kafka%20producer%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-1.jpg)



- Producer首先使用用户主线程将待发送的消息封装进一个ProducerRecord类实例中。
- 进行interceptor、序列化、分区等其他操作，发送到Producer程序中RecordAccumulator中。
- Producer的另一个工作线程（即Sender线程），则负责实时地从该缓冲区中提取出准备好的消息封装到一个批次的内，统一发送给对应的broker中。



## 消息发送源码分析

sender发送的时机是由两个指标决定的，一个是时间`linger.ms`，一个是数据量大小 `batch.size`

sender线程主要代码

run->run(long now)->sendProducerData

```java
    private long sendProducerData(long now) {
        Cluster cluster = metadata.fetch();
        //result.nextReadyCheckDelayMs表示下次检查是否ready的时间，也是//selecotr会阻塞的时间
        RecordAccumulator.ReadyCheckResult result = this.accumulator.ready(cluster, now);
        if (!result.unknownLeaderTopics.isEmpty()) {
            for (String topic : result.unknownLeaderTopics)
                this.metadata.add(topic);

            log.debug("Requesting metadata update due to unknown leader topics from the batched records: {}",
                result.unknownLeaderTopics);
            this.metadata.requestUpdate();
        }

        Iterator<Node> iter = result.readyNodes.iterator();
        long notReadyTimeout = Long.MAX_VALUE;
        while (iter.hasNext()) {
            Node node = iter.next();
            if (!this.client.ready(node, now)) {
                iter.remove();
                notReadyTimeout = Math.min(notReadyTimeout, this.client.pollDelayMs(node, now));
            }
        }

        // create produce requests
        Map<Integer, List<ProducerBatch>> batches = this.accumulator.drain(cluster, result.readyNodes, this.maxRequestSize, now);
        addToInflightBatches(batches);
        if (guaranteeMessageOrder) {
            // Mute all the partitions drained
            for (List<ProducerBatch> batchList : batches.values()) {
                for (ProducerBatch batch : batchList)
                    this.accumulator.mutePartition(batch.topicPartition);
            }
        }

        accumulator.resetNextBatchExpiryTime();
        List<ProducerBatch> expiredInflightBatches = getExpiredInflightBatches(now);
        List<ProducerBatch> expiredBatches = this.accumulator.expiredBatches(now);
        expiredBatches.addAll(expiredInflightBatches);

        if (!expiredBatches.isEmpty())
            log.trace("Expired {} batches in accumulator", expiredBatches.size());
        for (ProducerBatch expiredBatch : expiredBatches) {
            String errorMessage = "Expiring " + expiredBatch.recordCount + " record(s) for " + expiredBatch.topicPartition
                + ":" + (now - expiredBatch.createdMs) + " ms has passed since batch creation";
            failBatch(expiredBatch, -1, NO_TIMESTAMP, new TimeoutException(errorMessage), false);
            if (transactionManager != null && expiredBatch.inRetry()) {
                transactionManager.markSequenceUnresolved(expiredBatch.topicPartition);
            }
        }
        sensors.updateProduceRequestMetrics(batches);
       // 暂且只关心result.nextReadyCheckDelayMs
        long pollTimeout = Math.min(result.nextReadyCheckDelayMs, notReadyTimeout);
        pollTimeout = Math.min(pollTimeout, this.accumulator.nextExpiryTimeMs() - now);
        pollTimeout = Math.max(pollTimeout, 0);
        if (!result.readyNodes.isEmpty()) {
            log.trace("Nodes with data ready to send: {}", result.readyNodes);

            pollTimeout = 0;
        }
        sendProduceRequests(batches, now);
        return pollTimeout;
    }

```

```java
long pollTimeout = sendProducerData(now);
// poll最终会调用selector,pollTimeout也就是selector阻塞的时间
 client.poll(pollTimeout, now);
```



selector

```java
    /**
     * Check for data, waiting up to the given timeout.
     *
     * @param timeoutMs Length of time to wait, in milliseconds, which must be non-negative
     * @return The number of keys ready
     */
    private int select(long timeoutMs) throws IOException {
        if (timeoutMs < 0L)
            throw new IllegalArgumentException("timeout should be >= 0");

        if (timeoutMs == 0L)
            return this.nioSelector.selectNow();
        else
            return this.nioSelector.select(timeoutMs);
    }
```



```java
    public ReadyCheckResult ready(Cluster cluster, long nowMs) {
        Set<Node> readyNodes = new HashSet<>();
        // 初始化为最大值
        long nextReadyCheckDelayMs = Long.MAX_VALUE;
        Set<String> unknownLeaderTopics = new HashSet<>();

        boolean exhausted = this.free.queued() > 0;
        for (Map.Entry<TopicPartition, Deque<ProducerBatch>> entry : this.batches.entrySet()) {
            TopicPartition part = entry.getKey();
            Deque<ProducerBatch> deque = entry.getValue();

            Node leader = cluster.leaderFor(part);
            synchronized (deque) {
                if (leader == null && !deque.isEmpty()) {
                    unknownLeaderTopics.add(part.topic());
                } else if (!readyNodes.contains(leader) && !isMuted(part, nowMs)) {
                    ProducerBatch batch = deque.peekFirst();
                    if (batch != null) {
                        long waitedTimeMs = batch.waitedTimeMs(nowMs);
                        boolean backingOff = batch.attempts() > 0 && waitedTimeMs < retryBackoffMs;
                        // 和linger.ms有关
                        long timeToWaitMs = backingOff ? retryBackoffMs : lingerMs;
                        boolean full = deque.size() > 1 || batch.isFull();
                        boolean expired = waitedTimeMs >= timeToWaitMs;
                        boolean sendable = full || expired || exhausted || closed || flushInProgress();
                        if (sendable && !backingOff) {
                            readyNodes.add(leader);
                        } else {
                            long timeLeftMs = Math.max(timeToWaitMs - waitedTimeMs, 0);
                            nextReadyCheckDelayMs = Math.min(timeLeftMs, nextReadyCheckDelayMs);
                        }
                    }
                }
            }
        }
        return new ReadyCheckResult(readyNodes, nextReadyCheckDelayMs, unknownLeaderTopics);
    }
```

我们可以从实例化一个新的KafkaProducer开始分析（还没有调用send方法），在sender线程调用accumulator#ready(..)时候，会返回result，其中包含selector可能要阻塞的时间。由于还没有调用send方法，所以Deque<RecordBatch>为空，所以result中包含的nextReadyCheckDelayMs也是最大值，这个时候selector会一直阻塞。

然后我们调用send方法,该方法会调用waitOnMetadata，waitOnMetadata会调用`sender.wakeup()`，所以会唤醒sender线程。

这个时候会唤醒阻塞在selector#select(..)的sender线程，sender线程又运行到accumulator#ready(..)，当Deque<ProducerBatch>有值，所以返回的result包含的nextReadyCheckDelayMs不再是最大值，而是和linger.ms有关的值。也就是时候selector会最多阻塞lingger.ms后就返回，然后再次轮询。（根据源码分析，第一条消息会创建batch,因此newBatchCreated为true,同样会触发唤醒sender）

如果有一个ProducerBatch满了，也会调用Sender#wakeup(..)，

KafkaProducer#doSend(...)

```java
 if (result.batchIsFull || result.newBatchCreated) {
                log.trace("Waking up the sender since topic {} partition {} is either full or getting a new batch", record.topic(), partition);
                this.sender.wakeup();
            }
```



所以综上所述：只要满足linger.ms和batch.size满了就会激活sender线程来发送消息。

## 客户端顺序性保证

因为消息发送是批量发送的，那么就有可能存在上一批发送失败，接下来一批发送成功。即会出现数据乱序。

`max.in.flight.requests.per.connection=1`,限制客户端在单个连接上能够发送的未响应请求的个数。如果业务对顺序性有很高的要求，将该参数值设置为1。
## 总结

kafka producer大概流程如上，数据发送是**批量**的，最后是按Node发送响应的消息的。即其中Integer即为broker id。消息在RecordAccumulator中还是按TopicPartition存放的，但是在最终发送会作相应的转换。

sender发送的时机是由两个指标决定的，一个是时间`linger.ms`，一个是数据量大小 `batch.size`

了解producer发送流程，这样就可以让我们更改的生产消息，使用更多的特性，例如拦截器可以做各种特殊处理的场景。

## 参考

## 1. [kafka producer的batch.size和linger.ms](https://www.cnblogs.com/set-cookie/p/8902340.html)