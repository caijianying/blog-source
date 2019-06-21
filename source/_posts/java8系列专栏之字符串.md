---
title: java8系列专栏之字符串
tags:
  - 笔记
originContent: ''
categories:
  - java
toc: false
date: 2019-04-28 19:07:00
---

## 前言
写这篇文章的目的是为了记录一下学习笔记，其次为了能够在复习的时候快速掌握相关知识。本篇记录java8系列专栏之字符串
## 正文
### StringJoiner
##### 详解
拼接字符串
##### 用法
```
//不指定前缀和后缀
StringJoiner stringJoiner = new StringJoiner(",");
//指定前缀和后缀
//StringJoiner stringJoiner = new StringJoiner(",","{","}");
List<String> list = Arrays.asList("a","b","c");
list.forEach(str->stringJoiner.add(str));
```
### String.join
##### 详解
拼接字符串，缺点是无法指定前缀和后缀
##### 用法
```
List<String> list = Arrays.asList("a","b","c");
System.out.println(String.join(",", list));
```
## 参考
1. [Java8（2）：JDK 对字符串连接的改进](https://www.jianshu.com/p/0654d0b69eef)