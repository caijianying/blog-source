---
title: nodejs邮件发送
date: 2017-09-17 12:25:48
tags: 开发笔记
categories:
- node.js
---
# nodejs邮件发送

## 介绍
node.js发送邮件有多种第三方模块，比较有名的是[emailjs](https://github.com/eleith/emailjs),[nodemailer](https://github.com/nodemailer/nodemailer),个人感觉emailjs更轻量级，使用更简单，nodemailer关注人更多，功能更加完善。
## 实践
本次选用这两种，仅实现简单发送邮件功能，更多功能，请查询官网API.
首先新建文件夹node,然后在该文件夹下新建package.json文件，在其中增加以下配置
```
{
  "name": "node_email_demo",
  "description": "application created by truman",
  "version": "1.0.0",
  "dependencies": {
    "emailjs": "^1.0.8",
    "nodemailer": "^4.1.0"
  },
  "repository": "",
  "license": "MIT",
  "engines": {
    "node": ">=6.0.0"
  },
  "readmeFilename": "README.md"
}
```

在node 目录下执行命令

```
npm install
```
- emailjs
新建myemailjs.js文件，添加如下内容：
```
var email       = require("emailjs");
var server      = email.server.connect({
    user:    "aibibang@sohu.com",
    password:"*********",
    host:    "smtp.sohu.com",
    ssl:     false
});

// send the message and get a callback with an error or details of the message that was sent
 server.send({
     text:    "i hope this works",
     from:    "aibibang@sohu.com",
     to:      "aibibang@sohu.com",
     subject: "testing emailjs"
     }, function(err, message) { console.log(err || message); });
```

执行即可发送成功！
```
node myemailjs.js
```
- nodemailer
新建mynodemailer.js文件，添加如下内容：

```
const nodemailer=require("nodemailer");

let transporter = nodemailer.createTransport({
    host: 'smtp.sohu.com',
    port: 25,
    auth: {
            user: "aibibang@sohu.com", // generated ethereal user
            pass: "*********"  // generated ethereal password
    },
    secure: false // true for 465, false for other ports
});
var message = {
    from: 'aibibang@sohu.com',
    to: 'aibibang@sohu.com',
    subject: 'nodemailer test',
    text: 'i hope this works'
};

transporter.sendMail(message, function(err){
    if(err){
        console.log(err);
    }
});
```

执行即可发送成功！
```
node mynodemailer.js
```
## 总结
1. 如果SMPT服务器没有进行安全校验，那么一定要去掉用户与密码，这点要和java API区别，切记，为了这个问题，花费了一天代价。
例如如下问题：
```
{ Error: no form of authorization supported
    at module.exports (/data/truman/node/node_modules/emailjs/smtp/error.js:2:13)
    at initiate (/data/truman/node/node_modules/emailjs/smtp/smtp.js:543:44)
    at caller (/data/truman/node/node_modules/emailjs/smtp/smtp.js:48:14)
    at attempt (/data/truman/node/node_modules/emailjs/smtp/smtp.js:415:14)
    at caller (/data/truman/node/node_modules/emailjs/smtp/smtp.js:48:14)
    at response (/data/truman/node/node_modules/emailjs/smtp/smtp.js:345:13)
    at caller (/data/truman/node/node_modules/emailjs/smtp/smtp.js:48:14)
    at response (/data/truman/node/node_modules/emailjs/smtp/smtp.js:201:11)
    at caller (/data/truman/node/node_modules/emailjs/smtp/smtp.js:48:14)
    at Socket.response (/data/truman/node/node_modules/emailjs/smtp/smtp.js:181:11)
  code: 7,
```
2. 对于本地是否可以访问SMTP服务器，可以使用telnet,进行查验，如果都无法telnet通，肯定是无法发送邮件的。
```
telnet smtp.sohu.com 25
```