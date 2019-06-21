---
title: jvm 在docker中内存占用问题探索
date: 2018-07-22 18:35:47
tags: 开发笔记
categories:
         - java
         - docker
---
# jvm 在docker中内存占用问题探索

## 问题背景

最近有个项目在PRD上部署，因为涉及到读取大量数据，会出现内存占用。为了避免因为该项目影响线上其他服务，所以设置了-m=2048，结果发现运行会超过这个值，docker 进程即将该container killed. 

随后设置了好几个级别，直到-m=6048,依然无法避免container 被干掉。但是在本地测试和在同事机器上测试，不会出现内存飙升。同样的数据，同样的容器，唯一不同的就是机器物理配置的不同。


## 问题原因

线上机器是128g内存，目前制作的jre image 是1.8版本，未设置堆栈等jvm 配置，那么jvm 会字节分配一个默认堆栈大小，这个大小是根据物理机配置分配的。这样就会造成越高的配置，默认分配（**使用1/4的物理内存**）的堆内存就越大，而docker设置限制内存大小，jvm却无法感知，不知道自己在容器中运行。目前存在该问题的不止jvm,一些linux 命令也是如此，例如：top,free,ps等。

因此就会出现container 被docker killed情况。这是个惨痛教训。。。


## 问题复现

略...

有个不错的[文章](http://www.linux-ren.org/thread/89699.html)，可以查看一下。

## 解决方案

- **Dockerfile增加jvm参数**

在调用java 可以增加jvm 参数，控制堆栈大小。
```
CMD java  $JAVA_OPTIONS -jar java-container.jar
```

```
$ docker run -d --name mycontainer8g -p 8080:8080 -m 800M -e JAVA_OPTIONS='-Xmx300m' rafabene/java-container:openjdk-env
```
- **选用Fabric8 docker image**

镜像fabric8/java-jboss-openjdk8-jdk使用了脚本来计算容器的内存限制，并且使用50%的内存作为上限。也就是有50%的内存可以写入。你也可以使用这个镜像来开/关调试、诊断或者其他更多的事情
