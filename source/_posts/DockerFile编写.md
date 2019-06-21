---
title: DockerFile编写
date: 2016-10-24 21:07:56
tags: 虚拟化
categories:
- docker
---
所有指令都是大写
1. ADD,COPY

两个都是将本地文件复制到镜像中，区别是ADD可以指定绝对路径的文件，言外之意是可以上传除当前目录之外的文件。而COPY只能上传当前目录的文件。

这两条命令复制文件夹的话，只会讲子目录复制到指定目录下。例如

ADD  redis3.0.4 /opt/app/redis/

只会将redis3.0.4下文件复制到redis目录下，不包含redis3.0.4目录。COPY同理
2. CMD,ENTRYPOINT

两个都是容器启动时运行的命令，区别是CMD可以被覆盖，而ENTRYPOINT不会。ENTRYPOINT只能是最后一个生效。