---
title: docker registry创建
date: 2016-10-24 21:26:53
tags: 虚拟化
categories:
- docker
---
centos6.X版本，docker version1.7.1版本创建过程如下：
1. 运行容器
```
$ docker run -d -p 5000:5000 registry
```
2. 修改配置

在docker1.3.X版本以后，与docker registry交互默认使用的是https，此处需要修改为http。在/etc/sysconfig/docker文件中添加以下内容即可：
```
other_args="$other_args --insecure-registry myregistry.example.com:5000 "
```
在1.12最新版本中可以使用以下方式修改，原理是一致的。
```
 Create or modify /etc/docker/daemon.json
{ "insecure-registries":["myregistry.example.com:5000"] }
```
然后重新启动
```
$ service docker restart
```
3. 测试新的registry
- 镜像打标签
```
$ docker tag <img_id> myregistry.example.com:5000/truman/opentsdb
```
- 提交镜像
```
$ docker push  myregistry.example.com:5000/truman/opentsdb
```
然后通过docker images即可查看到push 的镜像

在别的机器中就可以拉取镜像了命令如下：
```
docker pull myregistry.example.com:5000/truman/opentsdb
```