---
title: nginx 教程之 nginx 配置学习
date: 2018-01-15 19:26:39
tags: 开发笔记
categories:
- nginx
---
# nginx 教程之 nginx 配置学习

## 1.location配置

语法规则： 
```
location [=|~|~*|^~] /uri/ { … }

= 表示精确匹配,这个优先级也是最高的

~  表示区分大小写的正则匹配

~* 表示不区分大小写的正则匹配(和上面的唯一区别就是大小写)

^~ 表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）。

!~和!~* 分别为区分大小写不匹配及不区分大小写不匹配的正则

/ 通用匹配，任何请求都会匹配到，默认匹配.
```

优先级=>^~>

首先匹配 =，其次匹配^~, 其次是按文件中顺序的正则匹配，最后是交给 / 通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求


一般线上的配置
```
location ~* .*\.(js|css)?$
{
        expires 7d; //7天过期,后续讲解
        access_log off; //不保存日志
}
 
location ~* .*\.(png|jpg|gif|jpeg|bmp|ico)?$
{        
        expires 7d;
        access_log off;
}
```

## 2.nginx 逻辑运算

nginx的配置中不支持if条件的逻辑与&& 逻辑或|| 运算 ，而且不支持if的嵌套语法，否则会报下面的错误：nginx: [emerg] invalid condition。
我们可以用变量的方式来间接实现。
要实现的语句：
```
if ($arg_unitid = 42012 && $uri ~/thumb/){
 echo "www.baidu.com";
}
```
可以这么来实现，如下所示：
```
set $flag 0;
if ($uri ~ ^/thumb/[0-9]+_160.jpg$){
 set $flag "${flag}1";
}
if ($arg_unitid = 42012){
 set $flag "${flag}1";
}
if ($flag = "011"){
 echo "www.baidu.com";
}
```

## 3.ngx_http_core_module模块提供的变量

参数名称 注释
```
$arg_PARAMETER HTTP 请求中某个参数的值，如/index.php?site=www.ttlsa.com，可以用$arg_site取得www.ttlsa.com这个值.
$args HTTP 请求中的完整参数。例如，在请求/index.php?width=400&height=200 中，$args表示字符串width=400&height=200.
$binary_remote_addr 二进制格式的客户端地址。例如：\x0A\xE0B\x0E
$body_bytes_sent 表示在向客户端发送的http响应中，包体部分的字节数
$content_length 表示客户端请求头部中的Content-Length 字段
$content_type 表示客户端请求头部中的Content-Type 字段
$cookie_COOKIE 表示在客户端请求头部中的cookie 字段
$document_root 表示当前请求所使用的root 配置项的值
$uri 表示当前请求的URI，不带任何参数
$document_uri 与$uri 含义相同
$request_uri 表示客户端发来的原始请求URI，带完整的参数。$uri和$document_uri未必是用户的原始请求，在内部重定向后可能是重定向后的URI，而$request_uri 永远不会改变，始终是客户端的原始URI.
$host 表示客户端请求头部中的Host字段。如果Host字段不存在，则以实际处理的server（虚拟主机）名称代替。如果Host字段中带有端口，如IP:PORT，那么$host是去掉端口的，它的值为IP。$host 是全小写的。这些特性与http_HEADER中的http_host不同，http_host只取出Host头部对应的值。 
$hostname 表示 Nginx所在机器的名称，与 gethostbyname调用返回的值相同 
$http_HEADER 表示当前 HTTP请求中相应头部的值。HEADER名称全小写。例如，示请求中 Host头部对应的值 用 $http_host表 
$sent_http_HEADER 表示返回客户端的 HTTP响应中相应头部的值。HEADER名称全小写。例如，用 $sent_ http_content_type表示响应中 Content-Type头部对应的值 
$is_args 表示请求中的 URI是否带参数，如果带参数，$is_args值为 ?，如果不带参数，则是空字符串 
$limit_rate 表示当前连接的限速是多少，0表示无限速 
$nginx_version 表示当前 Nginx的版本号 
$query_string 请求 URI中的参数，与 $args相同，然而 $query_string是只读的不会改变 
$remote_addr 表示客户端的地址 
$remote_port 表示客户端连接使用的端口 
$remote_user 表示使用 Auth Basic Module时定义的用户名 
$request_filename 表示用户请求中的 URI经过 root或 alias转换后的文件路径 
$request_body 表示 HTTP请求中的包体，该参数只在 proxy_pass或 fastcgi_pass中有意义 
$request_body_file 表示 HTTP请求中的包体存储的临时文件名 
$request_completion 当请求已经全部完成时，其值为 “ok”。若没有完成，就要返回客户端，则其值为空字符串；或者在断点续传等情况下使用 HTTP range访问的并不是文件的最后一块，那么其值也是空字符串。
$request_method 表示 HTTP请求的方法名，如 GET、PUT、POST等 
$scheme 表示 HTTP scheme，如在请求 https://nginx.com/中表示 https 
$server_addr 表示服务器地址 
$server_name 表示服务器名称 
$server_port 表示服务器端口 
$server_protocol 表示服务器向客户端发送响应的协议，如 HTTP/1.1或 HTTP/1.0
```