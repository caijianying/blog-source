---
title: kafka幂等性和事务使用及实现原理
tags:
  - kafka
  - 专栏
originContent: ''
categories:
  - kafka
toc: false
date: 2019-07-08 19:13:41
---

# kafka幂等性和事务使用及实现原理

## 开篇

在开始这篇之前，先抛出问题，这章解决如下问题：

1. 如何开启幂等性？
2. 如何使用事务？
3. 幂等性的原理
4. 事务实现原理

## 正文

### Producer 幂等性

Producer 的幂等性指的是当发送同一条消息时，数据在 Server 端只会被持久化一次，数据不丟不重，但是这里的幂等性是有条件的：

- 只能保证 Producer 在单个会话内不丟不重，如果 Producer 出现意外挂掉再重启是无法保证的（幂等性情况下，是无法获取之前的状态信息，因此是无法做到跨会话级别的不丢不重）;
- 幂等性不能跨多个 Topic-Partition，只能保证单个 partition 内的幂等性，当涉及多个 Topic-Partition 时，这中间的状态并没有同步。

如果需要跨会话、跨多个 topic-partition 的情况，需要使用 Kafka 的事务性来实现。

使用方式：`props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true");`

当幂等性开启的时候acks即为all。如果显性的将acks设置为0，-1，那么将会报错`Must set acks to all in order to use the idempotent producer. Otherwise we cannot guarantee idempotence.`

示例：

```java
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092");
props.put(ProducerConfig.ACKS_CONFIG, "all");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true");

KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(props);
kafkaProducer.send(new ProducerRecord<String, String>("truman_kafka_center", "1", "hello world.")).get();
kafkaProducer.close();
```

### 幂等性原理

幂等性是通过两个关键信息保证的，PID(Producer ID)和sequence numbers。

- PID 用来标识每个producer client
- sequence numbers 客户端发送的每条消息都会带相应的 sequence number，Server 端就是根据这个值来判断数据是否重复

producer初始化会由server端生成一个PID,然后发送每条信息都包含该PID和sequence number，在server端，是按照partition同样存放一个sequence numbers 信息，通过判断客户端发送过来的sequence number与server端number+1差值来决定数据是否重复或者漏掉。

通常情况下为了保证数据顺序性，我们可以通过`max.in.flight.requests.per.connection=1`来保证，这个也只是针对单实例。在kafka2.0+版本上，只要开启幂等性，不用设置这个参数也能保证发送数据的顺序性。

### 为什么要求 MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION 小于等于5

其实这里，要求 MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION 小于等于 5 的主要原因是：Server 端的 ProducerStateManager 实例会缓存每个 PID 在每个 Topic-Partition 上发送的最近 5 个batch 数据（这个 5 是写死的，至于为什么是 5，可能跟经验有关，当不设置幂等性时，当这个设置为 5 时，性能相对来说较高，社区是有一个相关测试文档），如果超过 5，ProducerStateManager 就会将最旧的 batch 数据清除。

假设应用将 MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION 设置为 6，假设发送的请求顺序是 1、2、3、4、5、6，这时候 server 端只能缓存 2、3、4、5、6 请求对应的 batch 数据，这时候假设请求 1 发送失败，需要重试，当重试的请求发送过来后，首先先检查是否为重复的 batch，这时候检查的结果是否，之后会开始 check 其 sequence number 值，这时候只会返回一个 OutOfOrderSequenceException 异常，client 在收到这个异常后，会再次进行重试，直到超过最大重试次数或者超时，这样不但会影响 Producer 性能，还可能给 Server 带来压力（**相当于client 狂发错误请求**）。

### Kafka 事务性

#### 示例

```java
//Producer
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true");
props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "transactional_id-0");
KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(props);
kafkaProducer.initTransactions();
kafkaProducer.beginTransaction();
for (int i = 0; i < 10; i++) {
    kafkaProducer.send(new ProducerRecord<String, String>("truman_kafka_center", "key"+i, "hello world.")).get();
}
kafkaProducer.commitTransaction();
kafkaProducer.close();
//Consumer
Properties config = new Properties();
config.put("group.id", "test11");
config.put("bootstrap.servers", "127.0.0.1:9092");
config.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
config.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
config.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");

KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(config);
consumer.subscribe(Arrays.asList(TOPIC));
boolean isConsumer = true;
while (isConsumer) {
    ConsumerRecords<String, String> records = (ConsumerRecords<String, String>) consumer
        .poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println("consumer message: key =" + record.key() + " value:" + record.value());
    }
}
consumer.close();
}
```

#### 事务实现原理

###### (1)查找TransactionCoordinator

通过transaction_id 找到TransactionCoordinator，具体算法是`Utils.abs(transaction_id.hashCode %transactionTopicPartitionCount )`，获取到partition，再找到该partition的leader,即为TransactionCoordinator。

###### (2)获取PID
凡是开启幂等性都是需要生成PID(Producer ID),只不过未开启事务的PID可以在任意broker生成，而开启事务只能在TransactionCoordinator节点生成。这里只讲开启事务的情况，Producer Client的`initTransactions()`方法会向TransactionCoordinator发起InitPidRequest ，这样就能获取PID。这里面还有一些细节问题，这里不探讨，例如transaction_id 之前的事务状态什么的。但需要说明的一点是这里**会将 transaction_id  与相应的 TransactionMetadata 持久化到事务日志**（_transaction_state）中。

###### (3)开启事务

Producer调用`beginTransaction`开始一个事务状态，这里只是在客户端将本地事务状态转移成 IN_TRANSACTION，只有在发送第一条信息后，TransactionCoordinator才会认为该事务已经开启。

###### (4)Consume-Porcess-Produce Loop

这里说的是一个典型的`consume-process-produce`场景：

```java
while (true) {
    ConsumerRecords records = consumer.poll(Duration.ofMillis(1000));
    producer.beginTransaction();
    //start
    for (ConsumerRecord record : records){
        producer.send(producerRecord(“outputTopic1”, record));
        producer.send(producerRecord(“outputTopic2”, record));
    }
    producer.sendOffsetsToTransaction(currentOffsets(consumer), group);
    //end
    producer.commitTransaction();
}
```
该阶段主要经历以下几个步骤：
1. AddPartitionsToTxnRequest
2. ProduceRequest
3. AddOffsetsToTxnRequest
4. TxnOffsetsCommitRequest

关于这里的详细介绍可以查看参考链接，或者直接查看官网文档！

###### (5)提交或者中断事务

Producer 调用 `commitTransaction()` 或者 `abortTransaction()` 方法来 commit 或者 abort 这个事务操作。

基本上经历以下三个步骤，才真正结束事务。

1. EndTxnRequest
2. WriteTxnMarkerRquest
3. Writing the Final Commit or Abort Message

其中EndTxnRequest是在Producer发起的请求，其他阶段都是在TransactionCoordinator端发起完成的。WriteTxnMarkerRquest是发送请求到partition的leader上写入事务结果信息（ControlBatch）,第三步主要是在`_transaction_state`中标记事务的结束。




## 参考

1.[Kafka 事务性之幂等性实现](http://matt33.com/2018/10/24/kafka-idempotent/)

2.[Kafka Exactly-Once 之事务性实现](http://matt33.com/2018/11/04/kafka-transaction/)

3[KIP-98 - Exactly Once Delivery and Transactional Messaging](https://cwiki.apache.org/confluence/display/KAFKA/KIP-98+-+Exactly+Once+Delivery+and+Transactional+Messaging)

