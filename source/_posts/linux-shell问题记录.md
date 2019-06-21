---
title: linux shell问题记录
date: 2017-08-21 17:15:16
tags: 开发笔记
categories:
- shell
---
最近做docker image,编写shell 脚本，遇到以下问题，做个记录

1.问题一：
```
if [ "${1:0:1}" = '-' ]; then
	set -- elasticsearch "$@"
fi
```
**解释**：以上if 条件是指第一个参数的第一个字符为`-`,则符合条件

2.问题二：
```
exec "$@"
```
**解释**：exec执行命令

3.问题三：
```
set -- elasticsearch "$@"
```
**解释**：set设置环境，这句话的意思其实是将elasticsearch 作为第一个参数补充到"$@"中
例如：
demo.sh
```
#!/bin/bash
set -- elasticsearch "$@"
echo "$@"
```
执行脚本
```
bash demo.sh hello truman
```
输出结果为：elasticsearch hello truman

4.问题四：
```
"$(id -u)"
```
**解释**：输出当前shell 环境UID(用户ID)
