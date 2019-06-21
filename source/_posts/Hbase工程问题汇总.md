---
title: Hbase工程问题汇总
date: 2016-04-06 22:11:35
tags: 大数据
categories:
- hbase
---
1.Hbase web整合报以下错误
SEVERE: Servlet.service() for servlet [jsp] in context with path [] threw exception [java.lang.AbstractMethodError: javax.servlet.jsp.JspFactory.getJspApplicationContext(Ljavax/servlet/ServletContext;)Ljavax/servlet/jsp/JspApplicationContext;] with root cause 
java.lang.AbstractMethodError: javax.servlet.jsp.JspFactory.getJspApplicationContext(Ljavax/servlet/ServletContext;)Ljavax/servlet/jsp/JspApplicationContext; 
**解决方案**
pom文件中排出冲突jar
```xml
  <!-- hbase -->
 <dependency>
  <groupId>org.apache.hbase</groupId>
  <artifactId>hbase-client</artifactId>
  <version>1.0.0-cdh5.4.0</version>
  <exclusions>
   <exclusion>
    <artifactId>jasper-compiler</artifactId>
    <groupId>tomcat</groupId>
   </exclusion>
   <exclusion>
    <artifactId>jasper-runtime</artifactId>
    <groupId>tomcat</groupId>
   </exclusion>
  </exclusions>
 </dependency>
```
***
2.Hbase jar引入，pom报错误Missing artifact jdk.tools:jdk.tools:jar:1.7
** 解决方案**
eclipse.ini (before -vmargs!):
<pre>
<code>
-vm
C:/{your_path_to_jdk170}/jre/bin/server/jvm.dll
</code>
</pre>
出现该错误的原因是eclipse bug ,eclipse打开使用的是jre 不是jdk下面的jre ,因此未找到tools.jar
***
3.eclipse java api 访问hbase 失败
** 解决方案**
   * zookeeper工作是否正常
   * 防火墙是否关闭



