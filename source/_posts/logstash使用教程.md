---
title: logstash使用教程
date: 2016-10-24 21:26:00
tags: research
categories:
- elasticsearch
---
## 简介
logstash是一个实时流水式开源数据收集引擎。具有强大的plugin。可以根据自己的业务场景选择不同的input filter output。绝大多数情况下都是结合ElasticSearch Kibana一起使用的，俗称ELK。
## 模块介绍
Logstash使用管道方式进行日志的搜集处理和输出。有点类似*NIX系统的管道命令 xxx | ccc | ddd，xxx执行完了会执行ccc，然后执行ddd。

在logstash中，包括了三个阶段:

输入input --> 处理filter（不是必须的） --> 输出output
### 配置文件说明
前面介绍过logstash基本上由三部分组成，input、output以及用户需要才添加的filter，因此标准的配置文件格式如下：
```
input {
   
}
filter {
    
}
output {
    
}
```
### 执行说明
```
bin/logstash -f demo.conf
```
## 使用Demo
### Output plugins ElasticSearch
案例使用如下：
```
output {
#stdout {  codec => rubydebug }
elasticsearch {
hosts => ["127.0.0.1:9200"]
template_overwrite => true
index => "rediscluster-%{+YYYY.MM.dd}"
workers => 5
}
}
```
### Output plugins opentsdb
使用logstash收集数据，并发送到opentsdb中。分为三部分：Input,Filter,Output

输入数据时，输入一条数据，回车。以下为三条测试数据：
```
threads.ThreadCount 1352279077 67 host=server1 port=1006
gc.PSScavenge.CollectionTime 1352279137 1360 host=server2 port=1010
memorypool.CodeCache.Usage_used 1352279137 11625472 host=server1 port=1009
```

- Input采用命令行输入数据
```
input {
   stdin{
   }
}
```

- Filter过滤组织数据

采用的是grok插件，可以使用其他插件完成相同的目的
```
filter {
    grok { 
        match => { "message" => "%{DATA:metricName} %{NUMBER:unixtime} %{NUMBER:data} host=%{DATA:metricHost} port=%{NUMBER:port}" }  
        remove_field => [ "host" ]
    }
    
} 
```
**备忘**：

1. logstash输入数据自带host,@timestamp等自带，为了避免干扰存入opentsdb数据，此处特将隐含的host字段去掉。
2. DATA/NUMBER等实为grok自带的正则规则。
- Output输出数据

此处输出数据到opentsdb中，官方文档有误，详见[源码](https://github.com/logstash-plugins/logstash-output-opentsdb/blob/master/lib/logstash/outputs/opentsdb.rb)
```
output {
    stdout {  codec => rubydebug }#此处是为了将filter结果输出到控制台中
    opentsdb {
        host => '***.***.***.***'
        port => 4242
        metrics => [
            "%{metricName}",
            "%{data}",
            "host",
            "%{metricHost}",
            "port",
            "%{port}"
        ]
    }
}
```
**备忘**：

opentsdb输入信息格式为：put metric timestamp value tagname=tagvalue tag2=value2，在logstash-output-opentsdb插件metrics配置中默认已经输入timestamp，因此metrics需要配置的第一个参数为metricName，第二个参数为 value 之后依次为tagname,tagValue。

## 参考
1. https://www.elastic.co/guide/en/logstash/current/index.html