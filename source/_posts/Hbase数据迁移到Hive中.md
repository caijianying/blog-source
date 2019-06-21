---
title: Hbase数据迁移到Hive中
date: 2016-03-26 21:00:28
tags: 大数据
categories:
- hbase
---
# 背景

&emsp;hadoop体系中许多数据是存放在hbase中，为了便捷实用其中的数据，可以将数据迁移到hive中，通过hive sql 可以迅速便捷的获取并操作相应的数据，同时还可以实用hive sql实现一些mapreduce 操作。具体hive优势详见[**官网**](https://hive.apache.org/)。
# 目的

将hbase已存在的表数据迁移到hive中
# 操作步骤

## 版本环境

| soft       | version          |
| ------------- |:-------------:|
| hive    | hive-1.1.0-cdh5.4.0 |
| hbase      | hbase-1.0.0-cdh5.4.0|
## 准备

首先要确保HIVE_HOME/lib 下HBase的jar包的版本要和实际环境中HBase的版本一致，需要用HBASE_HOME/lib/目录下得jar包：
<pre><code>
hbase-client-1.0.0-cdh5.4.0.jar 
hbase-common-1.0.0-cdh5.4.0-tests.jar
hbase-common-1.0.0-cdh5.4.0.jar
hbase-protocol-1.0.0-cdh5.4.0.jar
hbase-server-1.0.0-cdh5.4.0.jar
htrace-core-3.0.4.jar
htrace-core-3.1.0-incubating.jar 
</code></pre>
复制到HIVE_HOME/lib 下
## 操作及测试
1. hbase现有数据表如下:
<pre><code>
hbase(main):002:0> scan 'truman'
ROW                        COLUMN+CELL
 row2                      column=personal:city, timestamp=1458635086205, value=beijing
 row2                      column=personal:name, timestamp=1458635086189, value=reason2
 row2                      column=professional:designation, timestamp=1458635086210, value=test
1 row(s) in 0.5910 seconds</code></pre>
2. 新建hive与hbase关联表hbase_table_1
<pre><code>
[root@127.0.0.1 hive-1.1.0-cdh5.4.0]$ bin/hive
Logging initialized using configuration in jar:file:/data/bigdata/hive-1.1.0-cdh5.4.0/lib/hive-common-1.1.0-cdh5.4.0.jar!/hive-log4j.properties
WARNING: Hive CLI is deprecated and migration to Beeline is recommended.
hive> CREATE external TABLE hbase_table_1(key int, value string) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,personal:name") TBLPROPERTIES ("hbase.table.name" = "truman");
OK
Time taken: 0.857 seconds</code></pre>
3. 即可在关联表中查询hbase中truman表的数据
<pre><code>
hive> show tables;
OK
hbase_table_1
Time taken: 0.272 seconds, Fetched: 1 row(s)
hive> select * from hbase_table_1;
OK
NULL    reason2
Time taken: 0.625 seconds, Fetched: 1 row(s)</code></pre>


# 总结
&emsp;可以通过关联表实现hbase数据迁移到hive中，同理，hive中数据也可以通过该表迁移到hbase中。
在新建关联表一定要注意，hbase表存在则使用external。如果不存在则可以不需要该关键字
# 参考

http://www.micmiu.com/bigdata/hive/hive-hbase-integration/