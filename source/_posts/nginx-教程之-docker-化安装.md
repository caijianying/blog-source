---
title: nginx 教程之 docker 化安装
date: 2018-01-15 19:26:21
tags: 开发笔记
categories:
- nginx
---
# nginx 教程之 docker 化安装

## 1.Why docker 

因为要依赖很多lib,安装复杂，提高了学习和使用的难度，所以推荐使用docker 部署，可以快速迁移，快速部署，更多优点详见[docker官网](https://hub.docker.com/)

## 2.部署

这里我们选择[官方镜像](https://hub.docker.com/_/nginx/),使用文档也可以在这个页面内找到。

使用完整nginx配置文件替换默认配置信息
```
 docker run --name=nginx --net=host -v /host/path/nginx.conf:/etc/nginx/nginx.conf -d nginx:1.13.7-alpine
```

当然还可以继承默认配置，这样使用

```
docker run --name=nginx --net=host -v /host/path/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx:1.13.7-alpine
```

## 3.nginx image默认配置


```

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

## 4.相关学习资源

- [网上最全面nginx教程（近100篇文章整理）](http://blog.csdn.net/jinchaoh/article/details/49450435)
