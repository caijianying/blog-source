---
title: 使用maven创建自定义archetype
date: 2016-08-28 19:28:16
tags: 开发笔记
categories:
- java
---
## 前言
在开发过程中经常需要新建工程，新建工程使用自带的archetype,往往不能满足项目开发需求，这就需要我们开发出自己的archetype。
## 实现
本次使用create-from-project来实现自定义archetype（方法至少两种）
#### 1.构建模板项目
首先使用eclipse创建一个新的maven  project，然后把配置好的一些公用的东西放到相应的目录下面 比如说会将一些常用的java代码存放到src/main/java目录下面；会将一些通用的配置文件放到src/main/resources目录下面；如果是javeEE工程，还会有一些jsp等等的文件存放到src/main/webapp目录下面
#### 2.pom.xml编辑
在pom.xml文件中添加以下内容
```
<build>
    <pluginManagement>
    	<plugins>
    		<plugin>
    			<groupId>org.apache.maven.plugins</groupId>
    			<artifactId>maven-archetype-plugin</artifactId>
    			<version>2.4</version>
    		</plugin>
    	</plugins>
    </pluginManagement>
</build>
```
#### 3.编译
在工程跟目录下运行maven命令
```
mvn archetype:create-from-project
```
然后会在target目录下面生成generated-sources目录，这个就是生成的 archetype
#### 4.安装/发布
 进入generated-sourced/archetype目录，运行maven命令：
```
    mvn install
```
这样就把自定义的archetype安装到本地仓库了。

archetype安装的地址是在maven安装目录下面的conf/settings.xml文件中指定的(<localRepository>字节)。

默认会在  ~/.m2  目录下面生成一个archetype-catalog.xml文件（和默认的settings.xml在同一个目录), 声明了该archetype的groupId、artifactId和其他属性。

因为Eclipse创建maven项目过程中，选择的“Default Local”指向的地址就是 ~/.m2，所以文件archetype-catalog.xml会被eclipse自动读取，使用eclipse创建maven项目的时候可以在"Default Local"一项中找到刚才自定义archetype名字。

> 安装到本地仓库中的archetype只可以被自己使用，如果想要共享，那么在第四步的时候使用deploy命令，不要使用install命令。

#### 5.卸载
如果想要卸载刚才安装的archetype，只需要将~/.m2目录下面的archetype-catalog.xml文件中对应的<archetype>字节段删掉，并且把本地仓库中相应groupId和artifactId下面的文件删掉就可以了。

## 备注
**问题**：eclipse中找不到自定义archetype?

首先查看自定义的版本是否是0.0.1-SNAPSHOT，如果是这个的话，需要勾选include snapshot archetypes

## 参考
1.http://my.oschina.net/wangrikui/blog/498807
2.http://blog.csdn.net/sxdtzhaoxinguo/article/details/46895013


