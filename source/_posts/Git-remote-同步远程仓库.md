---
title: Git remote 同步远程仓库
date: 2018-01-15 19:18:45
tags: 开发笔记
categories:
- git
---
# Git remote 同步远程仓库
在正常开发过程中，我们会fork别人的项目，然后在本地开发，随着时间推移，fork源代码会更新，但是就出现了问题，怎么和源项目保持更新？

## Git更新fork项目代码

1.添加远程仓库

添加：
```
git remote add source_respository_name https://github.com/TrumanDu/nav-dashboad.git
```
查看项目所有remote仓库名称：
```
$ git remote  
   origin  
   source_respository_name 
```

2.同步源仓库信息到本地

```
$ git remote update source_repository_name 
```

3.将源仓库信息merge到本地分支

```
$ git checkout branch_name  
$ git rebase source_repository_name/branch_name 
```

## GitHub开发协同流程

GitHub常用的开发协同流程为：

将别人的仓库fork成自己的origin仓库 → git clone origin仓库到本地 → 本地添加fork源仓库 → 工作前先git remote update下fork源保持代码较新 → coding → push回自己 → github上提出Push Request即可