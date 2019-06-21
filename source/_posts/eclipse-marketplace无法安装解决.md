---
title: eclipse marketplace无法安装解决
date: 2018-01-15 19:21:04
tags: 问题答疑
categories:
- 开发工具
---
# eclipse marketplace无法安装解决

## 问题简介

最近看到eclipse发布了oxygen,心里就痒痒了，想着升级一下新版本，体验一下新功能，下载以后想着安装一些生产力工具，但是发现竟然无法安装，要求我配置代理信息，并且报以下错误：
```
Proxy Authentication Required Error
```
 心里一下子就有一千万个草泥马，再返回之前4.6版本，发现，这个版本竟然可以安装。。。。
 
 ## 解决方案
 
 经过千辛万苦的google，别问我问啥没用baidu,因为没有卵用。
 
 终于在程序员神器网站stackoverflow找到解决方案。
 在eclipse.ini中增加以下内容：
 ```
 -Dorg.eclipse.ecf.provider.filetransfer.excludeContributors=org.eclipse.ecf.provider.filetransfer.httpclient4
 ```
 
 然后就可以欢快的下载各种插件了。
 
 
 ## 参考
 
 [stackoverflow](https://stackoverflow.com/questions/39576290/eclipse-neon-http-proxy-authentication-required-error)