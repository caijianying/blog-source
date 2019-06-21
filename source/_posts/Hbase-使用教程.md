---
title: Hbase 命令教程
date: 2016-06-01 21:54:24
tags: 大数据
categories:
- hbase
---
# Hbase学习
## Hbase shell操作
### 一、建表
1.新建不带namespace表
```
create 'testTable','t1','t2'
```
2.新建带namespace表
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
### 二、查表
- 列出所有表
```
list
```
- 浏览某个表
``` hbase
scan 'testTable'
```
### 三、数据操作
- 增加数据
```
put 'testTable','row1','t1:name','value1'
put 'testTable','row2','t1:name','value2'
put 'testTable','row3','t1:name','value3'
```
- 获取数据
```
get 'testTable','row1'
```
- 删除数据
```

```
## 其他
待完善
## 参考
1.https://hbase.apache.org/book.html
