---
title: HBase常用的操作一(shell)
date: 2016-06-14 20:27:16
tags: 大数据
categories:
- hbase
---
HBase – Hadoop Database，是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可在廉价PC Server上搭建起大规模结构化存储集群。
HBase访问接口如下:

1. Native Java API，最常规和高效的访问方式，适合Hadoop MapReduce Job并行批处理HBase表数据
2. HBase Shell，HBase的命令行工具，最简单的接口，适合HBase管理使用
3. Thrift Gateway，利用Thrift序列化技术，支持C++，PHP，Python等多种语言，适合其他异构系统在线访问HBase表数据
4. REST Gateway，支持REST 风格的Http API访问HBase, 解除了语言限制
5. Pig，可以使用Pig Latin流式编程语言来操作HBase中的数据，和Hive类似，本质最终也是编译成MapReduce Job来处理HBase表数据，适合做数据统计
6. Hive，当前Hive的Release版本尚没有加入对HBase的支持，在Hive 0.7.0中支持HBase，可以使用类似SQL语言来访问HBase
本文针对HBase Shell，做一个详细讲解。

<!-- more -->

在hbase安装目录下执行 bin/hbase shell 即可进入shell操作
#### 1. 一般命令
##### 1.1status
功能：查询服务器状态
使用：
```
 hbase(main):003:0> status
4 servers, 0 dead, 1.2500 average load
```
 
##### 1.2version
功能：查询HBase版本信息
使用：
```
 hbase(main):004:0> version
1.0.0-cdh5.4.0, rUnknown, Tue Apr 21 12:19:34 PDT 2015
```
##### 1.3 whoami
功能：查看连接的用户
#### 2. DDL命令
##### 2.1 Create创建表
功能：创建一个表。创建一个表时，不指定具体的列名，但要指定列族名。
使用：create ‘表名’,’列族名1’,’列族名2’
```
hbase(main):005:0> create 'truman_test','user'
0 row(s) in 1.3680 seconds

=> Hbase::Table - truman_test
```
 备注：
新建不带namespace表
```
create 'testTable','t1','t2'
```
新建带namespace表
- 首先建一个namespace
```
create_namespace 'truman'
```
- 其次再新建表
```
create_namespace 'truman：test','t1','t2'
```
备注：t1,t2为column family
```
create'reason:user_test','bs',{NUMREGIONS=>2,SPLITALGO=>'HexStringSplit'}

```

##### 2.2 disable失效表
功能：失效一个表。当需要修改表结构、删除表时，需要先执行此命令。
使用：
```
 hbase(main):007:0> disable 'truman_test'
0 row(s) in 1.2660 seconds

hbase(main):008:0> describe 'truman_test'
Table truman_test is DISABLED
truman_test
COLUMN FAMILIES DESCRIPTION
{NAME => 'user', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => '
FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0',
BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
1 row(s) in 0.0210 seconds
```
 
##### 2.3 enable使失效表有效
功能：使表有效。在失效表以后，需要执行此命令，以使得表可用。
使用：
```
 hbase(main):009:0> enable 'truman_test'
0 row(s) in 0.4320 seconds

hbase(main):010:0> describe 'truman_test'
Table truman_test is ENABLED
truman_test
COLUMN FAMILIES DESCRIPTION
{NAME => 'user', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => '
FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0',
BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
1 row(s) in 0.0260 seconds
```
 
##### 2.4 alter修改表结构
功能：修改表结构，包括新增列族、删除列族等
使用：
新增列族（记得在执行alter之前，要先disable表）
```
 hbase(main):011:0> disable 'truman_test'
0 row(s) in 1.2480 seconds

hbase(main):012:0> alter 'truman_test', 'order'
Updating all regions with the new schema...
1/1 regions updated.
Done.
0 row(s) in 1.2320 seconds

hbase(main):013:0> enable 'truman_test'
0 row(s) in 0.4060 seconds
hbase(main):014:0> describe 'truman_test'
Table truman_test is ENABLED
truman_test
COLUMN FAMILIES DESCRIPTION
{NAME => 'order', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS =>
'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0',
 BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
{NAME => 'user', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => '
FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0',
BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
2 row(s) in 0.0240 seconds
```
 
删除列族
```
 hbase(main):015:0> disable 'truman_test'
0 row(s) in 1.2970 seconds

hbase(main):017:0> alter 'truman_test',{NAME=>'order',method=>'delete'}
Unknown argument ignored for column family order:
Updating all regions with the new schema...
1/1 regions updated.
Done.
0 row(s) in 1.2100 seconds

hbase(main):018:0> enable 'truman_test'
0 row(s) in 0.4170 seconds
```
 
重命名列族
列族不能被重命名。重命名一个列族的通常途径是使用API创建一个有着期望名称的新的列族，然后将数据复制过去，最后再删除旧的列族。
##### 2.5 describe查看表结构
功能：查看表结构
使用：
```
 hbase(main):010:0> describe 'truman_test'
Table truman_test is ENABLED
truman_test
COLUMN FAMILIES DESCRIPTION
{NAME => 'user', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => '
FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0',
BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
1 row(s) in 0.0260 seconds
```
 
##### 2.6 list列举数据库中的所有表
功能：查看数据库中所有的表
使用：
```
hbase(main):019:0> list
TABLE
reason
test
truman_test
3 row(s) in 0.0090 seconds

=> ["reason", "test", "truman_test"]
```
 
##### 2.7 drop删除表
功能：删除指定的表
使用：
```
hbase(main):022:0> disable 'tttt'
0 row(s) in 1.2370 seconds

hbase(main):023:0> drop 'tttt'
0 row(s) in 0.2140 seconds
```
 
#### 3. DML命令
##### 3.1 put插入数据
功能：插入一条数据到指定的表中。对于同一个rowkey，如果执行两次put，则第二次被认为是更新操作。
使用：put ‘表名’,’列族名1:列名1’,’值’
```
hbase(main):006:0> put 'truman_test','row1','user:name','truman'
0 row(s) in 0.1470 seconds
```
 
##### 3.2 get获取数据
功能：获取数据
使用：
获取指定rowkey的指定列族指定列的数据
```
 hbase(main):027:0> get 'truman_test','row1','user:name'
COLUMN                     CELL
 user:name                 timestamp=1465268767543, value=truman
1 row(s) in 0.0180 seconds
```
 
获取指定rowkey的指定列族所有的数据
```
hbase(main):028:0> get 'truman_test','row1','user'
COLUMN                     CELL
 user:age                  timestamp=1465277742287, value=20
 user:name                 timestamp=1465268767543, value=truman
2 row(s) in 0.0150 seconds 
```
 
获取指定rowkey的所有数据
```
 hbase(main):029:0> get 'truman_test','row1'
COLUMN                     CELL
 user:age                  timestamp=1465277742287, value=20
 user:name                 timestamp=1465268767543, value=truman
2 row(s) in 0.0270 seconds
```
 
获取指定时间戳的数据
```
 hbase(main):029:0> get 'truman_test','row1'
COLUMN                     CELL
 user:age                  timestamp=1465277742287, value=20
 user:name                 timestamp=1465268767543, value=truman
2 row(s) in 0.0270 seconds

hbase(main):030:0> put 'truman_test','row1','user:age','21'
0 row(s) in 0.0090 seconds

hbase(main):031:0> get 'truman_test','row1'
COLUMN                     CELL
 user:age                  timestamp=1465277824052, value=21
 user:name                 timestamp=1465268767543, value=truman
2 row(s) in 0.0140 seconds

hbase(main):034:0> get 'truman_test','row1',{COLUMN=>'user:age',TIMESTAMP=>1465277742287}
COLUMN                     CELL
 user:age                  timestamp=1465277742287, value=20
1 row(s) in 0.0350 seconds
```
##### 3.3 Count计算表的行数
功能：计算表的行数
使用：
```
 hbase(main):035:0> count 'truman_test'
1 row(s) in 0.0370 seconds

=> 1
```
 
##### 3.4 put更新数据
详见3.1
##### 3.5 scan全表扫描数据
功能：扫描全表所有数据
使用：
```
 hbase(main):036:0> scan 'truman_test'
ROW                        COLUMN+CELL
 row1                      column=user:age, timestamp=1465277824052, value=21
 row1                      column=user:name, timestamp=1465268767543, value=truman
1 row(s) in 0.0310 seconds
```
 
##### 3.6 delete删除数据
功能：删除表中的数据
使用：
删除指定rowkey的指定列族的列名的数据
```
 hbase(main):037:0> delete 'truman_test','row1','user:name'
0 row(s) in 0.0610 seconds
```
 
删除指定rowkey的指定列族的数据
```
hbase(main):001:0> delete 'truman_test','row1','user'
0 row(s) in 0.3730 seconds
```
 
##### 3.7 deleteall删除整行数据
功能：删除整行数据
使用：
```
 hbase(main):003:0> deleteall 'truman_test','row1'
0 row(s) in 0.0080 seconds
```
 
##### 3.8 truncate删除全表数据
功能：删除表中所有的数据。正如你看到的，在HBase的help命令里并没有
使用：
```
 hbase(main):005:0> truncate 'truman_test'
Truncating 'truman_test' table (it may take a while):
 - Disabling table...
 - Truncating table...
0 row(s) in 1.8220 seconds
```
#### 4. HBase Snapshots
##### 4.1 新增snapshot
功能: 为指定表新增一个snapshot
使用：
```
hbase(main):001:0> snapshot 'truman', 'trumanSnapshot-1'
0 row(s) in 1.0820 seconds
```
##### 4.2 删除snapshot
功能: 删除一个snapshot
使用：
```
hbase(main):001:0> delete_snapshot 'trumanSnapshot-1'
```
##### 4.4 用snapshot创建一个新表
功能: Clone a table from snapshot，新表必须不存在。
使用：
```
hbase(main):007:0> clone_snapshot 'trumanSnapshot-1','truman_new'
0 row(s) in 1.0260 seconds
```
##### 4.3 查看所有的snapshot
功能: 
使用：
```
hbase(main):001:0> list_snapshots
SNAPSHOT                   TABLE + CREATION TIME
 trumanSnapshot-1          truman (Wed Jun 08 01:43:36 -0400 2016)
1 row(s) in 0.3410 seconds

=> ["trumanSnapshot-1"]
```
##### 4.4 根据snapshot恢复数据
功能: 恢复表数据
使用：执行命令之前，首先要确认表的状态是disable,恢复完成之后，表的状态要更改成enable

```
hbase(main):008:0> disable 'truman'
0 row(s) in 1.3080 seconds

hbase(main):009:0> restore_snapshot 'trumanSnapshot-1'
0 row(s) in 0.8200 seconds

hbase(main):011:0> enable 'truman'
0 row(s) in 0.4530 seconds
```
#### 5. 参考
1. https://hbase.apache.org/book.html
2. http://coderbase64.iteye.com/blog/2074601