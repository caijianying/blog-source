---
title: Kibana之sentinl slack使用教程
date: 2017-09-17 12:26:39
tags: research
categories:
- elasticsearch
---
# Kibana之sentinl slack使用教程
## 简介
[Slack](https://slack.com/)是一个团队协作沟通平台，至于它的强大，就不在这里多说了，sentinl 集成slack,可以将监控告警发送至其中，这个平台支持PC和APP，可以让开发运维人员实时获取监控服务状态。
## 申请slack
1. 注册账户
2. 点击右上方按钮创建workspace
3. 新建channel ,进入创建好的workspace，在左侧菜单栏创建，将该chanel设置成public
4. 集成[incoming-webhook](https://my.slack.com/services/new/incoming-webhook),获取Webhook URL。详细步骤详见[此处](https://www.elastic.co/guide/en/x-pack/current/actions-slack.html#configuring-slack)
## kibana配置
在kibana.yml配置文件中增加以下内容
```
sentinl:
  settings:
     slack:
       active: true
       username: truman
       hook: 'https://hooks.slack.com/services/******/*****/*****'
       channel: '#messagesend'

```
其中hook配置项为Webhook URL
## sentinl 配置slack action
在sentinl中增加slack action
```
"slack action": {
        "throttle_period": "0h0m1s",
        "slack": {
          "channel": "#messagesend",
          "message": "payload.hits.total:{{payload.hits.total}}",
          "stateless": false
        }
```