---
title: linux环境jdk安装及配置
date: 2016-04-15 20:09:44
tags: 大数据
categories:
- linux
---
1.下载jkd（ http://www.oracle.com/technetwork/java/javase/downloads/index.html）

- 对于32位的系统可以下载以下两个Linux x86版本（uname -a 查看系统版本）



- 264位系统下载Linux x64版本


2.安装jdk（这里以.tar.gz版本，32位系统为例）

安装方法参考http://docs.oracle.com/javase/7/docs/webnotes/install/linux/linux-jdk.html 

- 选择要安装java的位置，如/usr/目录下，新建文件夹java(mkdir java)

- 将文件jdk-7u40-linux-i586.tar.gz移动到/usr/java

- 解压：tar -zxvf jdk-7u40-linux-i586.tar.gz

- 删除jdk-7u40-linux-i586.tar.gz（为了节省空间）

至此，jkd安装完毕，下面配置环境变量

3.打开
```
/etc/profile（vim /etc/profile）
```
在最后面添加如下内容：
```
JAVA_HOME=/usr/java/jdk1.7.0_40
CLASSPATH=.:$JAVA_HOME/lib.tools.jar
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH
```
4.生效
```
source /etc/profile
```

5.验证是否安装成功：java -version