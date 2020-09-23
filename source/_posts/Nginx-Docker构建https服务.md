---
title: Nginx Docker构建https服务
tags:
  - 笔记
originContent: ''
categories:
  - nginx
toc: false
date: 2020-09-23 21:24:29
---

## 前言
本篇文章介绍生成一个自签名SSL证书以及使用Nginx docker 代理一个https服务。

SSL证书验证安全连接，有两种验证模式：

1. 仅客户端验证服务器的证书，客户端自己不提供证书；
2. 客户端和服务器都互相验证对方的证书。

显然第二种更安全，一般web采用第一种，比较简单。

## 创建自签名证书
### 创建步骤
1. 创建Key；
2. 创建签名请求；
3. 将Key的口令移除；
4. 用Key签名证书。

### 创建脚本
```
#!/bin/sh

# create self-signed server certificate:

read -p "Enter your domain [www.example.com]: " DOMAIN

echo "Create server key..."

openssl genrsa -des3 -out $DOMAIN.key 2048

echo "Create server certificate signing request..."

SUBJECT="/C=US/ST=Mars/L=iTranswarp/O=iTranswarp/OU=iTranswarp/CN=$DOMAIN"

openssl req -new -subj $SUBJECT -key $DOMAIN.key -out $DOMAIN.csr

echo "Remove password..."

mv $DOMAIN.key $DOMAIN.origin.key
openssl rsa -in $DOMAIN.origin.key -out $DOMAIN.key

echo "Sign SSL certificate..."

openssl x509 -req -days 3650 -in $DOMAIN.csr -signkey $DOMAIN.key -out $DOMAIN.crt

echo "TODO:"
echo "Copy $DOMAIN.crt to /etc/nginx/ssl/$DOMAIN.crt"
echo "Copy $DOMAIN.key to /etc/nginx/ssl/$DOMAIN.key"
echo "Add configuration in nginx:"
echo "server {"
echo "    ..."
echo "    listen 443 ssl;"
echo "    ssl_certificate     /etc/nginx/ssl/$DOMAIN.crt;"
echo "    ssl_certificate_key /etc/nginx/ssl/$DOMAIN.key;"
echo "}"
```
执行以上脚本会生成4个文件：
- www.example.com.crt：自签名的证书
- www.example.com.csr：证书的请求
- www.example.com.key：不带口令的Key
- www.example.com.origin.key：带口令的Key
- ### nginx配置

#### 核心配置
```
    server {
        listen       443 ssl;
        server_name  localhost;

        ssl_certificate      	www.example.com.crt;
        ssl_certificate_key  	www.example.com.key;

        location / {
			proxy_pass http://localhost:8866/;
        }
    }
```
#### 完整配置
```
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout  65;
    gzip  on;
	include /etc/nginx/conf.d/*.conf;
    index   index.html index.htm;

    server {
        listen       80;
        server_name  localhost;

        location / {
			proxy_pass http://localhost:8866/;
			proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
		}
    }

    server {
        listen       443 ssl;
        server_name  localhost;

        ssl_certificate      	www.example.com.crt;
        ssl_certificate_key  	www.example.com.key;

        location / {
			proxy_pass http://localhost:8866/;
			proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
			proxy_cookie_path / "/; httponly; secure; SameSite=NONE";
        }
    }
}
```

## 启动nginx容器
```
docker run --name my-custom-nginx-container -p 443:443 -v "F:\docker-workspace\nginx-demo\nginx.conf:/etc/nginx/nginx.conf:ro" -v "F:\docker-workspace\nginx-demo\www.example.com.crt:/etc/nginx/www.example.com.crt" -v "F:\docker-workspace\nginx-demo\www.example.com.key:/etc/nginx/www.example.com.key" -d nginx
```

## 参考
1. [给Nginx配置一个自签名的SSL证书](https://www.liaoxuefeng.com/article/990311924891552)