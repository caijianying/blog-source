---
title: Redis Cluster 批量操作实现
date: 2016-06-05 10:27:55
tags: nosql
categories:
- redis
---
## 前言
之前发过一篇[**RedisCluster构建批量操作探讨**](http://trumandu.github.io/2016/05/09/RedisCluster%E6%9E%84%E5%BB%BA%E6%89%B9%E9%87%8F%E6%93%8D%E4%BD%9C%E6%8E%A2%E8%AE%A8/)，这篇文章针对其中第2种方案，做一次实现。在使用redis cluster 过程中，经常会需要用到批量操作，本次探讨因此产生。
<!-- more -->
## 实现
原理：将key分批计算所在node,然后在单个node上执行pipeline,即可完成批量操作
### key 槽计算
因为redis cluster 设计原理，在处理key时，会将 CRC16(key) mod 16384 散列的分布在集群的node 上，因此，我们需要计算该key所在slot.(利用jedis中自带的算法)
```
int slot = JedisClusterCRC16.getSlot(key);
```
### node及slot获取
同样基于jedis(2.8.1),核心用法如下：
```
Jedis jedisNode = new Jedis(anyHostAndPort.getHost(), anyHostAndPort.getPort());
List<Object> list = jedisNode.clusterSlots();
```
该方法获取到整个集群中slot在node节点的分布。例如：0,5000，node.
通过以上方法整合，即可实现批量操作。

### 整合案例
以下是我自己写的一个set类型读取的批量操作的实现
```

private static Map<String, JedisPool> nodeMap = jedis.getClusterNodes();

private static TreeMap<Long, String> slotHostMap = getSlotHostMap(redisConf[0]);
/**
	 * 将key按slort分批整理
	 * @param keys
	 * @return
	 */
private static Map<JedisPool, List<String>> getPoolKeyMap(List<String> keys) {
	Map<JedisPool, List<String>> poolKeysMap = new LinkedHashMap<JedisPool, List<String>>();
	try {
		for (String key : keys) {

			int slot = JedisClusterCRC16.getSlot(key);

			//获取到对应的Jedis对象，此处+1解决临界问题
			Map.Entry<Long, String> entry = slotHostMap.lowerEntry(Long.valueOf(slot+1));

			JedisPool jedisPool = nodeMap.get(entry.getValue());

			if (poolKeysMap.containsKey(jedisPool)) {
				poolKeysMap.get(jedisPool).add(key);
			} else {
				List<String> subKeyList = new ArrayList<String>();
				subKeyList.add(key);
				poolKeysMap.put(jedisPool, subKeyList);
			}
		}
	} catch (Exception e) {
		e.getMessage();
	}
	return poolKeysMap;
}

/**
 * slort对应node
 * @param anyHostAndPortStr
 * @return
 */
private static TreeMap<Long, String> getSlotHostMap(String anyHostAndPortStr) {
	TreeMap<Long, String> tree = new TreeMap<Long, String>();
	String parts[] = anyHostAndPortStr.split(":");
	HostAndPort anyHostAndPort = new HostAndPort(parts[0], Integer.parseInt(parts[1]));
	try {
		Jedis jedisNode = new Jedis(anyHostAndPort.getHost(), anyHostAndPort.getPort());
		List<Object> list = jedisNode.clusterSlots();
		for (Object object : list) {
			List<Object> list1 = (List<Object>) object;
			List<Object> master = (List<Object>) list1.get(2);
			String hostAndPort = new String((byte[]) master.get(0)) + ":" + master.get(1);
			tree.put((Long) list1.get(0), hostAndPort);
			tree.put((Long) list1.get(1), hostAndPort);
		}
		jedisNode.close();
	} catch (Exception e) {

	}
	return tree;
}

/**
 * 批量获取set类型数据
 * @param keys
 * @return
 */
public static Map<String, Set<String>> batchGetSetData(List<String> keys) {
	if (keys == null || keys.isEmpty()) {
		return null;
	}
	Map<JedisPool, List<String>> poolKeysMap = getPoolKeyMap(keys);
	Map<String, Set<String>> resultMap = new HashMap<String, Set<String>>();
	for (Map.Entry<JedisPool, List<String>> entry : poolKeysMap.entrySet()) {
		JedisPool jedisPool = entry.getKey();
		List<String> subkeys = entry.getValue();
		if (subkeys == null || subkeys.isEmpty()) {
			continue;
		}
		//申请jedis对象
		Jedis jedis = null;
		Pipeline pipeline = null;
		List<Object> subResultList = null;
		try {
			jedis = jedisPool.getResource();
			pipeline = jedis.pipelined();

			for (String key : subkeys) {
				pipeline.smembers(key);
			}

			subResultList = pipeline.syncAndReturnAll();
		} catch (JedisConnectionException e) {
			e.getMessage();
		} catch (Exception e) {
			e.getMessage();
		} finally {
			if (pipeline != null)
				try {
					pipeline.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			//释放jedis对象
			if (jedis != null) {
				jedis.close();
			}
		}
		if (subResultList == null || subResultList.isEmpty()) {
			continue;
		}
		if (subResultList.size() == subkeys.size()) {
			for (int i = 0; i < subkeys.size(); i++) {
				String key = subkeys.get(i);
				Object result = subResultList.get(i);
				resultMap.put(key, (Set<String>) result);
			}
		} else {
			System.out.println("redis cluster pipeline error!");
		}
	}
	return resultMap;
}
```
## 参考
[1] cachecloud client中实现