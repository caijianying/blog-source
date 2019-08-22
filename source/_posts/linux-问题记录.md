---
title: linux 问题记录
date: 2016-04-13 20:32:48
tags: 大数据
categories:
- linux
---
## 一、hostname 修改详解
   1. 暂时性修改使用命令：
   ``` 
   hostname ******
   ```
   此命令不用重启，重新打开一个终端，即可看到修改。但是重启后失效（本人red hat5竟然不会）
   2. 永久性修改

    在1之后，修改 /etc/sysconfig/network
***
## 二、配置ssh
**本节参考：[网址](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)**
1. 生成公钥
```
$ssh-keygen 
```
2. 公钥分发到远程主机
```    　
$ ssh-copy-id user@host
```
***
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@ WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
df:a8:ff:b2:0d:f8:1c:a9:d9:f9:fa:39:29:9d:6f:42.
Please contact your system administrator.
Add correct host key in /root/.ssh/known_hosts to get rid of this message.
Offending key in /root/.ssh/known_hosts:13
RSA host key for lab3 has changed and you have requested strict checking.
Host key verification failed.
```
问题解决：
删除/root/.ssh/known_hosts:13行中的信息即可

***