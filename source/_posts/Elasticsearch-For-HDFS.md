---
title: Elasticsearch For HDFS
date: 2018-07-22 18:51:22
tags: research
categories:
- elasticsearch
---
# Elasticsearch For HDFS

## 1.前言
本文主要是探讨repository-hdfs插件，该插件的目的是可以将es的数据备份到HDFS中，并且可以从该文件恢复数据。
***
## 2.安装

1. 安装
   ```
   sudo bin/elasticsearch-plugin install repository-hdfs
   ```
   这里的安装是**在线安装**，当然还可以离线安装。例如：
   ```
   sudo bin/elasticsearch-plugin install file:///home/hadoop/elk/repository-hdfs.zip
   ```
   
2. 卸载
   ```
   sudo bin/elasticsearch-plugin remove repository-hdfs
   ```
***
## 3.使用
### 3.1创建仓库
- 创建仓库
```
PUT _snapshot/my_hdfs_repository
{
  "type": "hdfs",
  "settings": {
    "uri": "hdfs://namenode:8020/",
    "path": "elasticsearch/respositories/my_hdfs_repository"
  }
}
```

属性|描述
---|---
`uri`| hdfs url 地址. 例如: "hdfs://<host>:<port>/". (Required)
`path`|数据备份与加载地址. 例如: "path/to/file". (Required)
`load_defaults`|是否加载默认的Hadoop配置。 (Enabled by default)
`conf.<key>`|要添加到Hadoop配置的内联配置参数。 （可选）插件只能识别hadoop[core](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml)和[hdfs]((http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml))配置文件中面向客户端的属性。
`compress`|是否压缩元数据. (Disabled by default)
`chunk_size`|覆盖块大小. (Disabled by default)
`security.principal`|

- 查看所有的仓库
```
GET /_snapshot/_all
```

### 3.2备份数据
 - 快照所有index
   ```
   PUT /_snapshot/my_hdfs_repository/snapshot_all?wait_for_completion=false
   ```
 - 快照指定index
   ```
   PUT /_snapshot/my_hdfs_repository/snapshot_1?wait_for_completion=false
    {
      "indices": "index_1,index_2",
      "ignore_unavailable": true,
      "include_global_state": false
    }
   ```
   其中indices可以指定多个index,例如：index_1,index_2
 - 查看快照进度
   ```
   GET /_snapshot/my_hdfs_repository/snapshot_all/_status
   ```
 - 取消快照任务（删除快照）
   ```
   DELETE /_snapshot/my_hdfs_repository/snapshot_all
   ```
   这个可以删除一个存在的快照或者取消一个正在进行的快照。

### 3.3恢复数据
 - 恢复指定快照中所有index
   ```
   POST /_snapshot/my_hdfs_repository/snapshot_all/_restore
   ```
 - 恢复指定快照中的index
   ```
   POST /_snapshot/my_hdfs_repository/snapshot_all/_restore
    {
      "indices": "metricbeat-6.3.0-2018.07.18",
      "ignore_unavailable": true,
      "include_global_state": true,
      "rename_pattern": "metricbeat-(.+)",
      "rename_replacement": "allrestored_metricbeat-$1"
    }
   ```
   在这里可以设置恢复后新的index的名称，也可以和之前的保持一致（前提是集群中不存在该index）
 - 查看恢复进度
   ```
   GET index1,index2/_recovery?human
   GET _cat/recovery?v
   ```
 - 恢复任务设置
   ```
   POST /_snapshot/my_hdfs_repository/snapshot_2/_restore
    {
      "indices": "ec_gatewaymsg_jpf",
      "ignore_unavailable": true,
      "include_global_state": true,
      "index_settings": {
        "index.number_of_replicas": 0
      },
      "rename_pattern": "ec_gatewaymsg_(.+)",
      "rename_replacement": "restore_$1"
    }
   ```
   在恢复过程，可以修改部分index 的设置，但是分片数除外。
***
## 4.参考
1. [repository-hdfs](https://www.elastic.co/guide/en/elasticsearch/plugins/current/repository-hdfs.html#repository-hdfs)
2. [modules-snapshots](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/modules-snapshots.html)