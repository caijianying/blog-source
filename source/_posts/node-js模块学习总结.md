---
title: node.js模块学习总结
date: 2018-01-15 19:23:42
tags: 开发笔记
categories:
- node.js
---
# node.js模块学习总结

## 简介
主要是记录node.js模块学习总结，对于不是专业前端开发人员，往往不清楚到底nodejs比传统javascript多了哪些内容，针对自己自我学习，做个总结。

## 正文

首先推荐一个学习网站，觉得挺不错的
```
http://nqdeng.github.io/7-days-nodejs/
```
### 常识性
在开始学习之前，有三个新增关键字需要理解一下：

---
1. **require**
>require函数用于在当前模块中加载和使用别的模块，传入一个模块名，返回一个模块导出对象
2. **exports**
>exports对象是当前模块的导出对象，用于导出模块公有方法和属性。别的模块通过require函数使用当前模块时得到的就是当前模块的exports对象
3. **module**
>通过module对象可以访问到当前模块的一些相关信息，但最多的用途是替换当前模块的导出对象
---

看到以上信息有个疑问，我们经常使用的是

code1
```
module.exports = function(){
     console.log('Hello World!');
}

```
和

code1
```
exports.hello = function () {
    console.log('Hello World!');
};
```
区别在哪里？

两点主要区别是在在于require在不同模块获取时，只是能获取该模块中的默认导出对象，即exports，只有通过exports.hello这种方式，即可获取该模块暴露的方法或者属性。而module.exports则是将默认的导出对象修改为一个函数。

### 内置对象


- assert - 断言
- Buffer - 缓冲器
- child_process - 子进程
- cluster - 集群
- console - 控制台
- crypto - 加密
- dgram - 数据报
- dns - 域名服务器
- Error - 异常
- events - 事件
- fs - 文件系统
- global - 全局变量
- http - HTTP
- https - HTTPS
- module - 模块
- net - 网络
- os - 操作系统
- path - 路径
- process - 进程
- querystring - 查询字符串
- readline - 逐行读取
- repl - 交互式解释器
- stream - 流
- string_decoder - 字符串解码器
- timer - 定时器
- tls - 安全传输层
- tty - 终端
- url - 网址
- util - 实用工具
- v8 - V8引擎
- vm - 虚拟机
- zlib - 压缩

试验的API
- async_hooks
- http2
- inspector
- napi
- perf_hooks

详细使用    [详见网址](http://nodejs.cn/api/)


