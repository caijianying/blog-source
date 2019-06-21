---
title: node.js读取json文件
date: 2017-09-17 12:27:15
tags: 开发笔记
categories:
- node.js
---
# node.js读取json文件
## 目的
在node.js项目中如何获取json文件，本篇文章主要讲述两个方面：1.客户端 2.服务端。

## 实践
本次采用web框架为express

1.客户端
目录结构为：
```
/
/public/conf/demo.json
/index.js
/package.json
/index.html
```
index.js内容如下：
```
var express = require('express');
var app = express();
app.use(express.static('public'));
app.listen(80, function () {
  console.log('Example app listening on port 80!');
});
```
在index.html中获取如下：
```
$.getJSON('conf/demo.json', function (data) {
			console.log(data);
	})
```

2.服务端
index.js内容如下：
```
var express = require('express');
var app = express();
var CONFPATH = "./public/conf/demo.json";
var fs = require('fs');
var result = JSON.parse(fs.readFileSync(CONFPATH));
	console.log(result);
app.listen(80, function () {
  console.log('Example app listening on port 80!');
});
```