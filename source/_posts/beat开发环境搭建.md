---
title: beat开发环境搭建
date: 2018-10-15 17:21:09
tags: 教程
categories:
- elasticsearch
---
# 前言
beats 是什么？Beats是您在服务器上作为代理安装的开源数据托运方，用于将操作数据发送到Elasticsearch。

elastc 官方提供的beats 有以下方面：

Audit data|Auditbeat
---|---
Log files|Filebeat
Availability|Heartbeat
Metrics|Metricbeat
Network traffic|Packetbeat
Windows event logs|Winlogbeat

![image](https://www.elastic.co/guide/en/beats/libbeat/current/images/beats-platform.png)
beats可以直接发送数据到Elasticsearch或通过Logstash，您可以在Kibana中进行可视化之前进一步处理和增强数据。

# 安装软件
1. 安装新版 [go](https://golang.org/dl/)

   beats 是用go 语言写的，因此需要安装go  
   (1).配置GOROOT 例如:<code>GOROOT=D:\Go\</code>
   (2).配置Path 环境中新增<code>D:\Go\bin</code>
   (3).配置GOPATH  GOPATH=<code>D:\Go\GOPATH</code>  
2. 安装 Python 2  

     (1).安装python  
     (2).安装pip   
     ```
     wget https://bootstrap.pypa.io/get-pip.py
     sudo python get-pip.py
     ```
     (3). virtualenv  
          virtualenv 用来生成支持make update
     ```
     pip install virtualenv
     ```
# 源码配置
    ```
         mkdir -p ${GOPATH}/src/github.com/elastic
         cd ${GOPATH}/src/github.com/elastic
         git clone https://github.com/elastic/beats.git
    ```
   **切记上面配置地址路径，不然运行就会报错**。

# 编译运行
1. 编译
   ```
   make collect
   make update
   make
   ```
2. 运行
   ```
   ./{beat} -e -d "*"
   ```
   ```* ```代表选择输出的debug 日志，例如```./metricset -e -d "kafka"``` 输出kafka moduel 相关debug log
# 注意事项

 执行```make update```可以生成配置文件和文档，例如运行beat需要field.yml文件，即可由该你命令生成，但是切记，该命令也会重写{beat}.yml配置文件  
 
# 参考
1. [beats-contributing](https://www.elastic.co/guide/en/beats/devguide/current/beats-contributing.html)

