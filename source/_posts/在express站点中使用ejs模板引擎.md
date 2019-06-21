---
title: 在express站点中使用ejs模板引擎
date: 2017-09-17 12:27:52
tags: 开发笔记
categories:
- node.js
---
# 在express站点中使用ejs模板引擎
选择ejs 是因为使用类似jsp技术，使用方式很像，页面可读性要比jade要好，个人习惯使用ejs。

## 配置模板
```
var express = require('express');

var app = express();
app.set('views', './views')
app.engine('.html', require('ejs').__express);
app.set('view engine', 'html');

```
## 后台代码

```
 app.use('/demo', function (req, res, next) {
    res.render('demo', {'template':"ejs});
  });
```


## 模板代码
demo.html
```
<!DOCTYPE html>
<html lang="en">
<body>
      hello <%=template %>
</body>
</html>

```