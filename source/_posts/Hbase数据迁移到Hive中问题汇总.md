---
title: Hbase数据迁移到Hive中问题汇总
date: 2016-03-26 20:29:05
tags: 大数据
categories:
- hbase
---
## 问题1
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. org/apach e/hadoop/hbase/mapreduce/TableInputFormatBase 

 ** 解决方法：**
***
将hbase/lib下
     hbase-common-1.0.0-cdh5.4.0-tests.jar
     hbase-common-1.0.0-cdh5.4.0.jar
copy到hive/lib下面
## 问题2
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaExcep tion(message:MetaException(message:java.io.IOException: java.lang.reflect.InvocationTargetExc eption
 ** 解决方法：**
***
将hbase/lib下
     htrace-core-3.0.4.jar
     htrace-core-3.1.0-incubating.jar 
copy到hive/lib下面
## 问题3
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:MetaException(message:Table truman already exists within HBase; use CREATE EXTERNAL TABLE instead to register it in Hive.)
 ** 解决方法：**
***
使用 external
例如：
<pre><code>
 create external table hive(id string,name string ,ct string)stored by   'org.apache.hadoop.hive.hbase.HBaseStorageHandler' with serdeproperties ("hbase.columns.mapping"= ":key,cf:name,cf:ct") tblproperties ("hbase.table.name" = "hbase");
</code></pre>
## 问题4 hive中建hbase关联表失败
** 解决方法：**
***
首先要确保<pre><code>HIVE_HOME/lib </code></pre>下HBase的jar包的版本要和实际环境中HBase的版本一致，需要用<pre><code>HBASE_HOME/lib/</code></pre>目录下得jar包：
<pre><code>
hbase-client-1.0.0-cdh5.4.0.jar 
hbase-common-1.0.0-cdh5.4.0-tests.jar
hbase-common-1.0.0-cdh5.4.0.jar
hbase-protocol-1.0.0-cdh5.4.0.jar
hbase-server-1.0.0-cdh5.4.0.jar
htrace-core-3.0.4.jar
htrace-core-3.1.0-incubating.jar 
</code></pre>
复制到<pre><code>HIVE_HOME/lib </code></pre>下。