---
title: Git命令总结
date: 2018-01-15 19:14:36
tags: 命令
categories:
- git
---
# Git Commant Summary

1.git checkout
- 撤销

使用-- filte 可以撤销文件修改到上一次commit或者add状态
```
git checkout -- readme.md
```
- 切换分支

直接跟分支名称，可以切换到指定分支上
```
git checkout master
```

2.git rebase

将指定分支合并到当前分支
```
git rebase origin
```

3.git remote

添加远程仓库
```
git add origin https://github.com/TrumanDu/nav-dashboad.git
```

4.git fetch

```
git fetch origin
```
丢弃本地改动与提交，从指定地方获取最新版代码到本地主分支

5.git reset

```
git reset --hard origin/master
```
丢弃本地改动与提交，从指定地方获取最新版代码到本地主分支