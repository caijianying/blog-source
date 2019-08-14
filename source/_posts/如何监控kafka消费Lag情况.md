---
title: 如何监控kafka消费Lag情况
tags:
  - 技术分享
originContent: ''
categories:
  - kafka
toc: false
date: 2019-04-13 19:35:10
---

## 前言
为什么会有这个需求？

kafka consumer 消费会存在延迟情况，我们需要查看消息堆积情况，就是所谓的消息Lag。目前是市面上也有相应的监控工具[KafkaOffsetMonitor](https://github.com/quantifind/KafkaOffsetMonitor)，我们自己也写了一套监控[kmanager](https://github.com/xaecbd/kmanager)。但是随着kafka版本的升级，消费方式也发生了很大的变化，因此，我们需要重构一下kafka offset监控。
## 正文

## 如何计算Lag
在计算Lag之前先普及几个基本常识

**LEO(LogEndOffset):** 这里说的和官网说的LEO有点区别，主要是指堆consumer可见的offset.即HW(High Watermark)

**CURRENT-OFFSET**: consumer消费到的具体位移

知道以上信息后，可知`Lag=LEO-CURRENT-OFFSET`。计算出来的值即为消费延迟情况。


## 官方查看方式
这里说的官方查看方式是在官网文档中提到的，使用官方包里提供的`bin/kafka-consumer-groups.sh`

最新版的工具只能获取到通过**broker**消费的情况

```
$ bin/kafka-consumer-groups.sh --describe --bootstrap-server 192.168.0.101:8092 --group test
Consumer group 'test' has no active members.

TOPIC              PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
truman_test_offset 2          1325            2361            1036            -               -               -
truman_test_offset 6          1265            2289            1024            -               -               -
truman_test_offset 4          1245            2243            998             -               -               -
truman_test_offset 9          1310            2307            997             -               -               -
truman_test_offset 1          1259            2257            998             -               -               -
truman_test_offset 8          1410            2438            1028            -               -               -
truman_test_offset 3          1225            2167            942             -               -               -
truman_test_offset 0          1218            2192            974             -               -               -
truman_test_offset 5          1262            2252            990             -               -               -
truman_test_offset 7          1265            2277            1012            -               -               -

```

## 程序查询方式
使用程序查询方式，有什么好处，可以实现自己的offset监控,可以无缝接入任何平台系统。

既然是用程序实现，那么做个更高级的需求，++**根据topic获取不同消费组的消费情况**++。

先定义两个实体

TopicConsumerGroupState
```
public class TopicConsumerGroupState {
	private String groupId;
	/**
	 * 消费方式：zk/broker 主要指的offset提交到哪里 新版本 broker 旧版本zk
	 */
	private String consumerMethod;
	private List<PartitionAssignmentState> partitionAssignmentStates;
	/**
	 * Dead：组内已经没有任何成员的最终状态，组的元数据也已经被coordinator移除了。这种状态响应各种请求都是一个response：
	 * UNKNOWN_MEMBER_ID Empty：组内无成员，但是位移信息还没有过期。这种状态只能响应JoinGroup请求
	 * PreparingRebalance：组准备开启新的rebalance，等待成员加入 AwaitingSync：正在等待leader
	 * consumer将分配方案传给各个成员 Stable：rebalance完成！可以开始消费了~
	 */
	private ConsumerGroupState consumerGroupState;

    //省略set和get方法..

}
```
PartitionAssignmentState
```
public class PartitionAssignmentState {
	private String group; 
	private String topic;
	private int partition;
	private long offset;
	private long lag;
	private String consumerId;
	private String host;
	private String clientId;
	private long logEndOffset;
	//省略set和get方法..
}

```

### broker消费方式 offset 获取
#### 实现思路
1. 根据topic 获取消费该topic的group
2. 通过使用KafkaAdminClient的describeConsumerGroups读取broker上指定group和topic的消费情况，可以获取到clientId,CURRENT-OFFSET,patition，host等
3. 通过consumer获取LogEndOffset（可见offset）
4. 将2与3处信息合并，计算Lag
#### 代码设计
引入最新版依赖,使用KafkaAdminClient获取1,2处信息
```
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.2.0</version>
</dependency>
```
1. 步骤1
```
	public Set<String> listConsumerGroups(String topic)
			throws InterruptedException, ExecutionException, TimeoutException {
		final Set<String> filteredGroups = new HashSet<>();

		Set<String> allGroups = this.adminClient.listConsumerGroups().all().get(30, TimeUnit.SECONDS).stream()
				.map(ConsumerGroupListing::groupId).collect(Collectors.toSet());
		allGroups.forEach(groupId -> {
			try {
				adminClient.listConsumerGroupOffsets(groupId).partitionsToOffsetAndMetadata().get().keySet().stream()
						.filter(tp -> tp.topic().equals(topic)).forEach(tp -> filteredGroups.add(groupId));
			} catch (InterruptedException e) {
				e.printStackTrace();
			} catch (ExecutionException e) {
				e.printStackTrace();
			}
		});
		return filteredGroups;
	}
```
2. 步骤2

使用describeConsumerGroup获取的消费情况，其中包含有members和无members情况。正常订阅消费是可以获取到members，但是通过制定patition消费拿不到members.
```
/**
	 * 根据topic 获取offset,该结果是不同group 组不同patition的当前消费offset
	 * 
	 * @param topic
	 * @return Map<String, Set<Entry<TopicPartition, OffsetAndMetadata>>>
	 * @throws InterruptedException
	 * @throws ExecutionException
	 * @throws TimeoutException
	 */
	public Map<String, Set<Entry<TopicPartition, OffsetAndMetadata>>> listConsumerGroupOffsets(String topic)
			throws InterruptedException, ExecutionException, TimeoutException {
		Set<String> groupIds = this.listConsumerGroups(topic);
		Map<String, Set<Entry<TopicPartition, OffsetAndMetadata>>> consumerGroupOffsets = new HashMap<>();
		groupIds.forEach(groupId -> {
			Set<Entry<TopicPartition, OffsetAndMetadata>> consumerPatitionOffsets = new HashSet<>();
			try {
				consumerPatitionOffsets = this.adminClient.listConsumerGroupOffsets(groupId)
						.partitionsToOffsetAndMetadata().get(30, TimeUnit.SECONDS).entrySet().stream()
						.filter(entry -> topic.equalsIgnoreCase(entry.getKey().topic())).collect(Collectors.toSet());
				;
			} catch (InterruptedException e) {
				e.printStackTrace();
			} catch (ExecutionException e) {
				e.printStackTrace();
			} catch (TimeoutException e) {
				e.printStackTrace();
			}
			consumerGroupOffsets.put(groupId, consumerPatitionOffsets);
		});
		return consumerGroupOffsets;
	}

	/**
	 * 根据topic 获取topicConsumerGroupStates,其中未包含lag/logEndOffset(consumer可见offset)
	 * 
	 * @param topic
	 * @return
	 * @throws InterruptedException
	 * @throws ExecutionException
	 * @throws TimeoutException
	 */
	public List<TopicConsumerGroupState> describeConsumerGroups(String topic)
			throws InterruptedException, ExecutionException, TimeoutException {
		final List<TopicConsumerGroupState> topicConsumerGroupStates = new ArrayList<>();
		Set<String> groupIds = this.listConsumerGroups(topic);
		Map<String, ConsumerGroupDescription> groupDetails = this.adminClient.describeConsumerGroups(groupIds).all()
				.get(30, TimeUnit.SECONDS);

		Map<String, Set<Entry<TopicPartition, OffsetAndMetadata>>> consumerPatitionOffsetMap = this
				.listConsumerGroupOffsets(topic);

		groupDetails.entrySet().forEach(entry -> {
			String groupId = entry.getKey();
			ConsumerGroupDescription description = entry.getValue();

			TopicConsumerGroupState topicConsumerGroupState = new TopicConsumerGroupState();
			topicConsumerGroupState.setGroupId(groupId);
			topicConsumerGroupState.setConsumerMethod("broker");
			topicConsumerGroupState.setConsumerGroupState(description.state());
			// 获取group下不同patition消费offset信息
			Set<Entry<TopicPartition, OffsetAndMetadata>> consumerPatitionOffsets = consumerPatitionOffsetMap
					.get(groupId);
			List<PartitionAssignmentState> partitionAssignmentStates = new ArrayList<>();

			if (!description.members().isEmpty()) {
				// 获取存在consumer(memeber存在的情况)
				partitionAssignmentStates = this.withMembers(consumerPatitionOffsets, topic, groupId, description);
			} else {
				// 获取不存在consumer
				partitionAssignmentStates = this.withNoMembers(consumerPatitionOffsets, topic, groupId);
			}
			topicConsumerGroupState.setPartitionAssignmentStates(partitionAssignmentStates);
			topicConsumerGroupStates.add(topicConsumerGroupState);
		});

		return topicConsumerGroupStates;
	}

	private List<PartitionAssignmentState> withMembers(
			Set<Entry<TopicPartition, OffsetAndMetadata>> consumerPatitionOffsets, String topic, String groupId,
			ConsumerGroupDescription description) {
		List<PartitionAssignmentState> partitionAssignmentStates = new ArrayList<>();
		Map<Integer, Long> consumerPatitionOffsetMap = new HashMap<>();
		consumerPatitionOffsets.forEach(entryInfo -> {
			TopicPartition topicPartition = entryInfo.getKey();
			OffsetAndMetadata offsetAndMetadata = entryInfo.getValue();
			consumerPatitionOffsetMap.put(topicPartition.partition(), offsetAndMetadata.offset());
		});
		description.members().forEach(memberDescription -> {
			memberDescription.assignment().topicPartitions().forEach(topicPation -> {
				PartitionAssignmentState partitionAssignmentState = new PartitionAssignmentState();
				partitionAssignmentState.setPartition(topicPation.partition());
				partitionAssignmentState.setTopic(topic);
				partitionAssignmentState.setClientId(memberDescription.clientId());
				partitionAssignmentState.setGroup(groupId);
				partitionAssignmentState.setConsumerId(memberDescription.consumerId());
				partitionAssignmentState.setHost(memberDescription.host());
				partitionAssignmentState.setOffset(consumerPatitionOffsetMap.get(topicPation.partition()));
				partitionAssignmentStates.add(partitionAssignmentState);
			});
		});
		return partitionAssignmentStates;
	}

	private List<PartitionAssignmentState> withNoMembers(
			Set<Entry<TopicPartition, OffsetAndMetadata>> consumerPatitionOffsets, String topic, String groupId) {
		List<PartitionAssignmentState> partitionAssignmentStates = new ArrayList<>();
		consumerPatitionOffsets.forEach(entryInfo -> {
			TopicPartition topicPartition = entryInfo.getKey();
			OffsetAndMetadata offsetAndMetadata = entryInfo.getValue();
			PartitionAssignmentState partitionAssignmentState = new PartitionAssignmentState();
			partitionAssignmentState.setPartition(topicPartition.partition());
			partitionAssignmentState.setTopic(topic);
			partitionAssignmentState.setGroup(groupId);
			partitionAssignmentState.setOffset(offsetAndMetadata.offset());
			partitionAssignmentStates.add(partitionAssignmentState);
		});
		return partitionAssignmentStates;
	}
```
3. 步骤3
```
	public Map<TopicPartition, Long>  consumerOffset (String gorupName,String topicName) {
		Properties consumerProps = new Properties();
		consumerProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.0.1.101:9092");
		consumerProps.put(ConsumerConfig.GROUP_ID_CONFIG, gorupName);
		consumerProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
		consumerProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		consumerProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,  StringDeserializer.class);
		consumerProps.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
		@SuppressWarnings("resource")
		KafkaConsumers<String, String>  consumer  = new KafkaConsumers<>(consumerProps);
		KafkaConsumer<String, String> kafkaConsumer = consumer.subscribe(topicName);
		List<PartitionInfo> patitions = kafkaConsumer.partitionsFor(topicName);
		List<TopicPartition>topicPatitions = new ArrayList<>();
		patitions.forEach(patition->{
			TopicPartition topicPartition = new TopicPartition(topicName,patition.partition());
			topicPatitions.add(topicPartition);
		});
		Map<TopicPartition, Long> result = kafkaConsumer.endOffsets(topicPatitions);
		return result;
	}
```
4. 步骤4
```
	private List<TopicConsumerGroupState> getBrokerConsumerOffsets(String clusterID, String topic) {
		List<TopicConsumerGroupState> topicConsumerGroupStates = new ArrayList<>();
		try {
			topicConsumerGroupStates = this.describeConsumerGroups(topic);
			// 填充lag/logEndOffset
			topicConsumerGroupStates.forEach(topicConsumerGroupState -> {
				String groupId = topicConsumerGroupState.getGroupId();
				List<PartitionAssignmentState> partitionAssignmentStates = topicConsumerGroupState
						.getPartitionAssignmentStates();
				Map<TopicPartition, Long> offsetsMap = this.consumerOffset(clusterID, groupId, topic);
				for (Entry<TopicPartition, Long> entry : offsetsMap.entrySet()) {
					long logEndOffset = entry.getValue();
					for (PartitionAssignmentState partitionAssignmentState : partitionAssignmentStates) {
						if (partitionAssignmentState.getPartition() == entry.getKey().partition()) {
							partitionAssignmentState.setLogEndOffset(logEndOffset);
							partitionAssignmentState.setLag(getLag(partitionAssignmentState.getOffset(), logEndOffset));
						}
					}
				}
			});
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		} catch (TimeoutException e) {
			e.printStackTrace();
		}
		return topicConsumerGroupStates;
	}
```
### zookeeper消费方式 offset 获取
#### 实现思路
1. 根据topic 获取消费该topic的group
2. 读取zookeeper上指定group和topic的消费情况，可以获取到clientId,CURRENT-OFFSET,patition。
3. 通过consumer获取LogEndOffset（可见offset）
4. 将2与3处信息合并，计算Lag
#### 代码设计
引入两个依赖,主要是为了读取zookeeper节点上的数据
```
		<dependency>
			<groupId>com.101tec</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.11</version>
		</dependency>
		<dependency>
			<groupId>org.apache.kafka</groupId>
			<artifactId>kafka_2.11</artifactId>
		</dependency>
```

1. 步骤1
```
public Set<String> listTopicGroups(String topic) {
		Set<String> groups = new HashSet<>();
		List<String> allGroups = zkClient.getChildren("/consumers");
		allGroups.forEach(group -> {
			if (zkClient.exists("/consumers/" + group + "/offsets")) {
				Set<String> offsets = new HashSet<>(zkClient.getChildren("/consumers/" + group + "/offsets"));
				if (offsets.contains(topic)) {
					groups.add(group);
				}
			}
		});
		return groups;
	}
```
2. 步骤2
```
public Map<String, Map<String, String>> getZKConsumerOffsets(String groupId, String topic) {
		Map<String, Map<String, String>> result = new HashMap<>();
		String offsetsPath = "/consumers/" + groupId + "/offsets/" + topic;
		if (zkClient.exists(offsetsPath)) {
			List<String> offsets = zkClient.getChildren(offsetsPath);
			offsets.forEach(patition -> {
				try {
					String offset = zkClient.readData(offsetsPath + "/" + patition, true);
					if (offset != null) {
						Map<String, String> map = new HashMap<>();
						map.put("offset", offset);
						result.put(patition, map);
					}
				} catch (Exception e) {
					e.printStackTrace();
				}

			});
		}
		String ownersPath = "/consumers/" + groupId + "/owners/" + topic;
		if (zkClient.exists(ownersPath)) {
			List<String> owners = zkClient.getChildren(ownersPath);
			owners.forEach(patition -> {
				try {
					try {
						String owner = zkClient.readData(ownersPath + "/" + patition, true);
						if (owner != null) {
							Map<String, String> map = result.get(patition);
							map.put("owner", owner);
							result.put(patition, map);
						}
					} catch (Exception e) {
						e.printStackTrace();
					}
				} catch (Exception e) {
					e.printStackTrace();
				}

			});
		}

		return result;
	}
```
3. 步骤3

略
4. 步骤4
```
	private List<TopicConsumerGroupState> getZKConsumerOffsets(String topic) {
		final List<TopicConsumerGroupState> topicConsumerGroupStates = new ArrayList<>();
		Set<String> zkGroups = this.listTopicGroups(topic);
		zkGroups.forEach(group -> {
			Map<String, Map<String, String>> zkConsumerOffsets = this.getZKConsumerOffsets(group,
					topic);

			Map<TopicPartition, Long> offsetsMap = this.consumerOffset(group, topic);

			TopicConsumerGroupState topicConsumerGroupState = new TopicConsumerGroupState();
			topicConsumerGroupState.setGroupId(group);
			topicConsumerGroupState.setConsumerMethod("zk");

			List<PartitionAssignmentState> partitionAssignmentStates = new ArrayList<>();

			Map<String, Long> offsetsTempMap = new HashMap<>(offsetsMap.size());
			offsetsMap.forEach((k, v) -> {
				offsetsTempMap.put(k.partition() + "", v);
			});

			zkConsumerOffsets.forEach((patition, topicDesribe) -> {
				Long logEndOffset = offsetsTempMap.get(patition);
				String owner = topicDesribe.get("owner");
				long offset = Long.parseLong(topicDesribe.get("offset"));
				PartitionAssignmentState partitionAssignmentState = new PartitionAssignmentState();
				partitionAssignmentState.setClientId(owner);
				partitionAssignmentState.setGroup(group);
				partitionAssignmentState.setLogEndOffset(logEndOffset);
				partitionAssignmentState.setTopic(topic);
				partitionAssignmentState.setOffset(offset);
				partitionAssignmentState.setPartition(Integer.parseInt(patition));
				partitionAssignmentState.setLag(getLag(offset, logEndOffset));

				partitionAssignmentStates.add(partitionAssignmentState);
			});
			topicConsumerGroupState.setPartitionAssignmentStates(partitionAssignmentStates);
			topicConsumerGroupStates.add(topicConsumerGroupState);
		});
		return topicConsumerGroupStates;
	}
```
## 参考
1. [如何获取Kafka的消费者详情——从Scala到Java的切换](https://blog.csdn.net/u013256816/article/details/79968647)
2. [Kafka的Lag计算误区及正确实现](https://blog.csdn.net/u013256816/article/details/79955578)
