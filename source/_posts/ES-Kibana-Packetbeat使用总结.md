---
title: ES+Kibana+Packetbeat使用总结
date: 2016-07-18 21:15:32
tags: research
categories:
- elasticsearch
---
# 简介
ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎。Kibanna是针对ElasticSearch一个界面UI。Packetbeat是监控网络数据包一个工具，分布式wireshark。

Packetbeat 是一个实时的网络数据包分析器，通过结合ES可以构建一个应用监控和性能分析系统。Packetbeat 可以将监控数据发送到es，或者redis,logstash中。目前支持以下协议：
- ICMP (v4 and v6)
- DNS
- HTTP
- Mysql
- PostgreSQL
- Redis
- Thrift-RPC
- MongoDB
- Memcache

# 搭建
## 搭建Es
1. 下载
```
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.4/elasticsearch-2.3.4.tar.gz
```
2. 解压
```
tar -xvf elasticsearch-2.3.4.tar.gz
```
3. 配置 ES
默认配置即可正常使用，但配置config/elasticsearch.yml使用效果更好。
配置如下：
```
 cluster.name: my-application
 node.name: node-1
 node.master: true
 node.data: true 
 path.logs: /data/truman/elasticsearch-2.3.4/logs
 bootstrap.mlockall: true

 network.host: 192.168.1.42

 http.port: 9200

 discovery.zen.ping.multicast.enabled: false
 discovery.zen.ping.unicast.hosts: ["192.168.1.42"]

 discovery.zen.fd.ping_interval: 1s
 discovery.zen.fd.ping_timeout: 30s
 discovery.zen.fd.ping_retries: 5

 action.auto_create_index: true
 action.disable_delete_all_indices: true
 indices.memory.index_buffer_size: 30%
 indices.memory.min_shard_index_buffer_size: 12mb
 indices.memory.min_index_buffer_size: 96mb
 action.write_consistency: one
 index.number_of_shards: 3
 index.number_of_replicas: 1
 threadpool:
  bulk:
   type: fixed
   queue_size: 300

 index.translog.flush_threshold_period: 10m
```
4. 启动
在使用root账户启动脚本可能会报错，在次修改一下elasticsearch启动脚本，增加以下内容：
```
ES_JAVA_OPTS="-Des.insecure.allow.root=true"
```
然后后台启动脚本
```
nohup bin/elasticsearch &
```
访问 http://192.168.1.42:9200/即可查看是否启动成功。
5. 安装head plugin
head 插件能够可视化操作index与数据,在http://192.168.1.42:9200/_plugin/head/上进行操作

安装
```
bin/plugin install mobz/elasticsearch-head
```
移除
```
bin/plugin remove head
```
## 安装kibana
1. 下载
```
wget https://download.elastic.co/kibana/kibana/kibana-4.5.2-linux-x64.tar.gz
```
2. 启动
```
tar -xvf kibana-4.5.2-linux-x64.tar.gz
cd kibana-4.5.2-linux-x64
bin/kibana
```
在浏览器中访问http://192.168.1.42:5601查看是否成功。
## 安装Packetbeat
1. 下载
```
wget https://download.elastic.co/beats/packetbeat/packetbeat-1.2.3-x86_64.tar.gz
```
2. 解压
```
tar -xvf packetbeat-1.2.3-x86_64.tar.gz
```
3. 配置 Packetbeat
修改es输出即可,默认是localhost,将该内容修改为es所在的主机ip，即可使用默认配置运行了。
```
output:
  ### Elasticsearch as output
  elasticsearch:
    # Array of hosts to connect to.
     hosts: ["192.168.1.42:9200"]
     template:

      # Template name. By default the template name is packetbeat.
      #name: "packetbeat"

      # Path to template file
      path: "packetbeat.template.json"

      # Overwrite existing template
      #overwrite: false
```
修改完成后，可以通过以下命令检验修改配置是否正确(该shell需要root权限)
```
sudo ./packetbeat -configtest -e
```
4. 加载索引模板（index template）到es
- 加载
```
curl -XPUT 'http://192.168.1.42:9200/_template/packetbeat' -d@/etc/packetbeat/packetbeat.template.json
```
加载成功后，访问以下网址，看packetbeat索引模板是否加上。
```
http://192.168.1.42:9200/_template/packetbeat
```
- 删除模板文件
```
curl -XDELETE 'http://192.168.1.42:9200/_template/packetbeat'
```
5. 启动
```
sudo nohup ./packetbeat &
```
6. 测试
模拟简单http请求
```
curl http://www.aibibang.com > /dev/null
```
查询数据
```
curl -XGET 'http://192.168.1.42:9200/packetbeat-*/_search?pretty'
```
查询出数据，即可证明安装成功！

## 经验总结
1. 索引未创建成功

在搭建过程中，可能由于索引模板配置错误，导致索引未创建成功。但还存在一种情况，即没有数据采集进去。采集到数据以后才会根据索引模板创建索引
2. thrift 监控配置

目前对thrift监控仅支持binary协议，详细配置如下
```
thrift:
    # Configure the ports where to listen for Thrift-RPC traffic. You can disable
    # the Thrift-RPC protocol by commenting out the list of ports.
    ports: [9090]
    transport_type: framed
    protocol_type: binary
    string_max_size: 200
    collection_max_size: 20
    capture_reply: true
    obfuscate_strings: true
    drop_after_n_struct_fields: 100
```
3. 修改采集字段
~~使用过程中，希望采集自定义字段，当前版本未发现~~。但在最新版本5.0.0-alpha4中可通过修改yml配置文件删除掉不需要采集的字段。
在5.0.0-alpha4中配置如下：
```
filters:
 - drop_fields:
     fields: ["request", "query","server","proc","client_server"]
```
其实老的版本在index template中也是可以设置的。通过设置_source,例如：
```
"_source": {
        "excludes": [
          "request", 
		  "query",
		  "server",
		  "proc",
		  "params",
		  "beat.hostname",
		  "client_server"
        ]
      },
```
