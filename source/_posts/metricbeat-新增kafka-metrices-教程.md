---
title: metricbeat 新增kafka metrices 教程
date: 2018-10-15 17:36:56
tags: 教程
categories:
- elasticsearch
---
# 项目背景
本次教程是编写metrices,开发moduel 基本差不多，可以参考[creating-metricbeat-module](creating-metricbeat-module)。

本次教程是新增kafka metrices ，增加filesize metrices,实现的功能是根据配置的kafka 数据文件目录，获取所有topic，不同patition 数据文件大小，将该数据收集到elasticsearch中，通过kibana 根据不同粒度监控kafka集群。
# 正文
## beats架构
![image](https://www.elastic.co/guide/en/beats/devguide/current/images/beat_overview.png)
## 项目生成
```
cd metricbeat
make create-metricset
```
根据提示输入相应的内容，然后生成field.yml文件(make update),编辑metricbeat.yml文件 ，编译然后运行即可。。
## 配置写入字段类型及文件
cd filesize

1. 编辑fields.yml
```
- name: filesize
  type: group
  description: >
    filesize
  fields:
    - name: topic
      type: keyword
      description: >
        topic
    - name: partition
      type: long
      description: >
        partition
    - name: filesize
      type: long
      description: >
        topic data file size       

```
2. 编辑docs.asciidoc

 略

## 读取配置
```
type MetricSet struct {
    mb.BaseMetricSet
	dataPath string
}

// New creates a new instance of the MetricSet. New is responsible for unpacking
// any MetricSet specific configuration options if there are any.
func New(base mb.BaseMetricSet) (mb.MetricSet, error) {
      // Unpack additional configuration options.
	  config := struct {
		DataPath string `config:"dataPath"`
		}{
			DataPath:"",
		}

        err := base.Module().UnpackConfig(&config)
		if err != nil {
				return nil, err
		}
	return &MetricSet{
		BaseMetricSet: base,
		dataPath: config.DataPath,
	}, nil
}
```

## 指标采集
```
func (m *MetricSet) Fetch(report mb.ReporterV2) {
    PthSep := string(os.PathSeparator)
	var dataPath = m.dataPath
    files, _ := ioutil.ReadDir(dataPath)
    for _, f := range files {
		if !f.IsDir() {
			continue
		}
        var path = dataPath+PthSep+f.Name()
		cfiles,_ := ioutil.ReadDir(path)
		var filesize = f.Size()

		for _, cf := range cfiles {
            filesize = filesize + cf.Size()
		}
        
		var name = f.Name();
		var index = strings.LastIndex(name,"-")
		if index <0 {
			continue
		}
		var topic = name[0:index]
		var partition = name[index+1:len(name)]
		debugf("topic:%v",f.Name())
		report.Event(mb.Event{
			MetricSetFields: common.MapStr{
				"topic": topic,
				"partition": partition,
                "filesize": filesize,
			},
		})
    }
}
```
## 完整代码
```
package filesize

import (
	"github.com/elastic/beats/libbeat/common"
	"github.com/elastic/beats/libbeat/common/cfgwarn"
	"github.com/elastic/beats/metricbeat/mb"
	"io/ioutil"
	"strings"
	"os"
	"github.com/elastic/beats/libbeat/logp"
)

// init registers the MetricSet with the central registry as soon as the program
// starts. The New function will be called later to instantiate an instance of
// the MetricSet for each host defined in the module's configuration. After the
// MetricSet has been created then Fetch will begin to be called periodically.
func init() {
	mb.Registry.MustAddMetricSet("kafka", "filesize", New)
}

var debugf = logp.MakeDebug("kafka")
// MetricSet holds any configuration or state information. It must implement
// the mb.MetricSet interface. And this is best achieved by embedding
// mb.BaseMetricSet because it implements all of the required mb.MetricSet
// interface methods except for Fetch.
type MetricSet struct {
    mb.BaseMetricSet
	dataPath string
}

// New creates a new instance of the MetricSet. New is responsible for unpacking
// any MetricSet specific configuration options if there are any.
func New(base mb.BaseMetricSet) (mb.MetricSet, error) {
      // Unpack additional configuration options.
	  config := struct {
		DataPath string `config:"dataPath"`
		}{
			DataPath:"",
		}

        err := base.Module().UnpackConfig(&config)
		if err != nil {
				return nil, err
		}
	return &MetricSet{
		BaseMetricSet: base,
		dataPath: config.DataPath,
	}, nil
}

// Fetch methods implements the data gathering and data conversion to the right
// format. It publishes the event which is then forwarded to the output. In case
// of an error set the Error field of mb.Event or simply call report.Error().
func (m *MetricSet) Fetch(report mb.ReporterV2) {
    PthSep := string(os.PathSeparator)
	var dataPath = m.dataPath
    files, _ := ioutil.ReadDir(dataPath)
    for _, f := range files {
		if !f.IsDir() {
			continue
		}
        var path = dataPath+PthSep+f.Name()
		cfiles,_ := ioutil.ReadDir(path)
		var filesize = f.Size()

		for _, cf := range cfiles {
            filesize = filesize + cf.Size()
		}
        
		var name = f.Name();
		var index = strings.LastIndex(name,"-")
		if index <0 {
			continue
		}
		var topic = name[0:index]
		var partition = name[index+1:len(name)]
		debugf("topic:%v",f.Name())
		report.Event(mb.Event{
			MetricSetFields: common.MapStr{
				"topic": topic,
				"partition": partition,
                "filesize": filesize,
			},
		})
    }
}

```
## 运行
1. 编译
   ```
   make collect
   make
   ```
2. 运行
   ```
   ./{beat} -e -d "*"
   ```
   <code>* </code>代表选择输出的debug 日志，例如<code>./metricset -e -d "kafka"</code> 输出kafka moduel 相关debug log
   
   
**tip**: 在field.yml有变化的时候，记得先执行<code>make update</code>,该命令会重写metricbeat.yml文件。

# 开发建议
可以使用如下代码做到debug日志
```
var debugf = logp.MakeDebug("kafka")
debugf("topic:%v",f.Name())
```
# 参考
1.[creating-metricsets](https://www.elastic.co/guide/en/beats/devguide/current/creating-metricsets.html)
