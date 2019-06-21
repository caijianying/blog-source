---
title: Eclipse Maven Update 时JDK版本变更问题
date: 2016-10-24 21:21:38
tags: 开发笔记
categories:
- java
---
1.新建一个Maven项目JDK版本和系统版本不对应，

2.右键Maven项目->Maven->Update ProjectJDK版本改变了，

3.操作系统的JDK重装了新的版本，这是引起前面两个现象的主要原因。

修改方法（假如系统jdk版本是1.8）：

方法一：在pom.xml文件中指定jdk的版本:
```
<build>  
        <plugins>  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-compiler-plugin</artifactId>  
                <version>3.1</version>  
                <configuration>  
                    <source>1.8</source>  
                    <target>1.8</target>  
                </configuration>  
            </plugin>  
        </plugins>  
    </build>
```    
方法二：修改settings.xml，找到profiles节点，在里面添加
```
<profile>    
    <id>jdk-1.8</id>    
     <activation>    
          <activeByDefault>true</activeByDefault>    
          <jdk>1.8</jdk>    
      </activation>    
<properties>    
<maven.compiler.source>1.8</maven.compiler.source>    
<maven.compiler.target>1.8</maven.compiler.target>    
<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>    
</properties>    
</profile> 
```
然后在update项目就可以保持和系统jdk版本一致。推荐使用第二种方法，减少对每个项目配置。