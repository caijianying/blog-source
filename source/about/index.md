---
title: 个人简介
date: 2016-03-12 21:11:34
---

专注于大数据领域（redis,kafka,elastic stack）/软件架构设计，内心怀储互联网之梦。相信代码改变世界，程序设计人生。

期待更多志同道合的朋友交流。


# 联系方式

- Email：1753177225@qq.com
- QQ：1753177225

---

# 个人信息

 - Truman/男/1990
 - 本科/西安邮电大学
 - 工作年限：6年
 - 技术博客：http://trumandu.github.io
 - Github:  http://github.com/TrumanDu
 - 期望职位：Java高级程序员，架构师
 - 期望薪资：税前月薪20k+，特别喜欢的公司可例外
 - 期望城市：西安

---

# 工作经历

## newegg公司 （ 2015年11月 ~ 至今 ）

### KafkaCenter
- **项目描述**：KafkaCenter 是kafka一站式平台，提供自助，监控，管理，运维，生态等全平台功能。
- **项目特色**：消费监控自研，丰富的kafka周边生态，强大完整的集群管理功能。
- **项目职责**: 主要负责架构设计，平台代码开发，项目管理等工作。
- **项目业绩**: 目前作为持newegg企业kafka支撑平台。

### RCT项目 
- **项目描述**：RCT 是一个通过解析rdb文件对redis内存结构分析的一站式平台。 支持对非集群/集群rdb文件分析、Slowlog查询与监控、ClientList查询与监控。
- **项目特色**：采用分布式架构，通过解析redis rdb文件，多角度多场景分析redis中内存数据结构，提供友好的可视化报表以及邮件通知等功能。
- **项目职责**: 主要负责架构设计，平台代码开发，项目管理等工作。
- **项目业绩**: 目前已在产线上使用，多次发挥重要作用，为公司节省了宝贵内存资源。

### SyncBigdataPlatform项目 
- **项目描述**：SyncBigdataPlatform 是依赖大数据存储技术，提供高性能读写，jumplocation,以及跨location数据同步服务平台。DB层采用Hbase,Cassandra,Cache层采用Redis，数据同步层采用Kafka。提供restful接口，以供业务调用。
- **项目特色**：支持自定义业务扩展，支持热部署，支持配置信息动态调整，支持可视化配置信息更改，支持多数据中心高速数据同步，支持jumplocation操作，支持业务数据格式与数据库scheme自定义设计
- **项目职责**: 主要负责架构设计，平台代码开发，项目管理等工作。
- **项目业绩**: 目前平台上线多个主要服务，可以支持重点核心业务系统，上线一年来，稳定运行无误，获得公司优秀项目荣誉。

### 基础服务搭建与运维
- **项目职责**: 搭建RedisCluster,Elasticsearch,KafkaCluster集群，对集群进行日常维护，处理线上问题，开发周边组件，便于更好的维护集群。
- **项目业绩**: 为newegg线上服务提供可靠稳定的cache,消息，检索服务。

### 服务Docker化
- **项目描述**：针对公司kafka/redis/elasticsearch/opentsdb等基础服务docker化，其次针对业务项目进行docker化部署
- **项目职责**: 针对不同需求场景设计部署架构，编写shell脚本，Dockfile文件。
- **项目业绩**: 将我们项目组所有服务实现全部docker化，降低服务维护难度，极大的解放劳动力。

### RedisClientAPI项目
- **项目描述**：该项目提供一个使用Redis的rest/thrift API,客户端可以跨多种编程语言，更高效，更便捷的使用Redis的功能。
- **项目职责**: 负责该项目的设计及开发工作
- **项目业绩**: 提供高效API服务，承受上亿次调用，未出现任何事故。实现了RedisCluster批量操作，慢查询追踪等功能。项目难点是对于批量操作的实现方案改造。

### MonitorPlatform项目 
- **项目描述**：该项目可以用来检测机器指标，基础服务指标，业务指标等数据，通过可视化技术快捷高效的监控相关服务。
- **项目职责**: 主要负责该项目的设计与开发工作，该项目利用ES stack技术，使用metricbeat收集机器指标信息，使用grafana做可视化。
- **项目业绩**: 实现我们团队服务的无死角化监控，打造具体我们team特色的监控平台。

### 其他项目
redis cluster监控，EC Dashboard，kibana 插件(email_table,indices_view),ServerLoginAlert,AnomalyDetection

## huatek公司 （ 2014年1月 ~ 2015年10月 ）

### 基于Activiti工作流引擎开发
- **项目描述**：公司发展拙见壮大，出差请假管理混乱，公司技术支持管理需要，同时业务组系统需要相关工作流引擎支持，在此背景下，公司专门成立人员开发工作流引擎。此系统可以可视化设计流程，快速便捷的适应不同业务场景需要。目前已经上线三套流程。该系统包含以下几个模块：自定义字段、自定义表单、流程设计、用户组管理、申请任务、待办任务、任务列表等模块。
- **项目职责**: 
负责工作流引擎表单设计，流程权限设计与开发，流程数据的汇总与导出，各个模块数据的查询等。
- **项目业绩**: 
顺利实现公司办公平台出差请假流程，技术支持流程。为固定资产业务系统提供购置流程。


### 李宁订单优化管理系统
- **项目描述**：由于目前SAP系统中根据合同开单时，需要大量人工计算，为了提高订单配货发单效率，以此为基础为李宁发货员工开发一套针对订单拆单、配货，其中包含一部分有规律有逻辑的计算法则，达到订单拆分的目的系统.该系统包含以下几个模块：信息管理、流程审批、分货管理、订单管理、报表信息。
- **项目职责**: 按照计划表参与开发该系统信息管理、流程审批、分货管理、订单管理、报表信息模块，接口模块。
- **项目业绩**: 按照项目需求，完成目前所涉及的所有开发任务。

---

# 开源项目和作品

## 开源项目

 - [KafkaCenter](https://github.com/xaecbd/KafkaCenter)主导设计及开发，维护，获得开源社区关注。
 - [redis-manager](https://github.com/ngbdf/redis-manager)(参与) redis 管理工具，获得业界关注。
 - [RCT](https://github.com/xaecbd/RCT) redis 内存分析工具，获得业界关注。
 - [autocomplate-shell](https://github.com/TrumanDu/autocomplate-shell)：VS Code 编写shell插件，下载量16k+
 - [indices_view](https://github.com/TrumanDu/indices_view)：查看ES indices kibana 插件已被[elastic](https://www.elastic.co/guide/en/kibana/current/known-plugins.html)官网收录，[码云周刊第68期](https://blog.gitee.com/2018/04/16/weekly068/)推荐 ）
 - [cleaner](https://github.com/TrumanDu/cleaner)：设置es index ttl 插件,已被[elastic](https://www.elastic.co/guide/en/kibana/current/known-plugins.html)官网收录

## 技术文章
- [技术博客](http://trumandu.github.io)
- [微信公众号](https://mp.weixin.qq.com/s/C6qIg9H6Og3AHzXJq9AYdQ)
- [RedisCluster构建批量操作探讨](http://trumandu.github.io/2016/05/09/RedisCluster%E6%9E%84%E5%BB%BA%E6%89%B9%E9%87%8F%E6%93%8D%E4%BD%9C%E6%8E%A2%E8%AE%A8/)
- [Kibana Plugin Development Tutorial](https://trumandu.gitbook.io/kibana-plugin-development-tutorial/) 
- [Java初级架构师入阶系列专栏](https://trumandu.gitbook.io/java-architect-tutorial/)

# 技能清单

以下均为我熟练使用的技能

- 语言： Java/Go/Bash/Node.js
- 前端框架：React/vue
- RPC框架：Thrift
- Web框架：Spring/Express
- ORM框架：Mybatis
- 微服务架构：Springboot
- 项目管理：Maven
- 关系型数据库：MySQL/SQLite
- NOSQL：Redis/Hbase/Cassandra/Elasticsearch
- 消息队列：Kafka
- 实时计算：KSQL
- 虚拟化：Docker
- 大数据相关：Hadoop/Hive/Zookeeper/Storm/Oozie
- 监控相关：Opentsdb/Elastic stack(es/kibana/logstash)
- 系统：Linux/Window
- 版本管理：SVN/GIT

---
