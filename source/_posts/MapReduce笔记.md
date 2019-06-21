---
title: MapReduce笔记
date: 2016-10-24 21:05:43
tags: 大数据
categories:
- hadoop
---
## MapReduce运行过程
个人理解整个过程是先对数据分片（这个过程还未读取真正数据），将数据划分到多个map,一个job可以包含多个map,MapReduce框架将多个job发送到多个节点上执行，每个job中map读取自己分片数据，然后根据业务代码过滤，再根据map输出进行reduce操作，最后将生成结果输出到一个目录中。

