---
title: git reset、git checkout和git revert
date: 2018-07-22 18:42:36
tags: 开发笔记
categories:
- git
---

### 命令场景总结

命令|作用域	|常用情景
---|---|--
git reset|提交层面|在私有分支上舍弃一些没有提交的更改
git reset|文件层面|将文件从缓存区中移除
git checkout|提交层面|切换分支或查看旧版本
git checkout|文件层面|舍弃工作目录中的更改
git revert|	提交层面|在公共分支上回滚更改

### 详细解释

1. git checkout
> 一般有两种场景：切换分支，舍弃工作目录中的更改

例如：
```
git checkout brach2
git checkout -- src/cli/serve/serve.js
```
2. git revert
>撤销一个提交的同时会创建一个新的提交。这是一个安全的方法，因为它不会重写提交历史。比如，下面的命令会找出倒数第二个提交，然后创建一个新的提交来撤销这些更改，然后把这个提交加入项目中。

```
git checkout hotfix
git revert HEAD~2
```
3. git reset
>如果你的更改还没有共享给别人，git reset是撤销这些更改的简单方法。当你开发一个功能的时候发现『糟糕，我做了什么？我应该重新来过！』时，reset就像是go-to命令一样。
除了在当前分支上操作，


- 提交层面

reset将一个分支的末端指向另一个提交。这可以用来移除当前分支的一些提交。你传入HEAD以外的其他提交的时候要格外小心，因为reset操作会重写当前分支的历史
```
git checkout hotfix
git reset HEAD~2
```
可以通过传入这些标记来修改你的缓存区或工作目录：

--soft – 缓存区和工作目录都不会被改变

--mixed – 默认选项。缓存区和你指定的提交同步，但工作目录不受影响

--hard – 缓存区和工作目录都同步到你指定的提交
- 文件层面

```
git reset HEAD~2 foo.py
```
会将当前的foo.py从缓存区中移除出去，而不会影响工作目录中对foo.py的更改。--soft、--mixed和--hard对文件层面的git reset毫无作用，因为缓存区中的文件一定会变化，而工作目录中的文件一定不变。



### 总结

一般我们开发中常用以下两个恢复文件

1. git checkout
```
git checkout -- src/cli/serve/serve.js
```
这条命令把serve.js从HEAD中签出并且把它恢复成未修改时的样子
2. git reset --hard HEAD
```
git reset --hard HEAD
```
这条命令会把你工作目录中所有未提交的内容清空(当然这不包括未置于版控制下的文件 untracked files).让工作目录回到上次提交时的状态(last committed state)
3. git revert
撤消(revert)了前期修改的提交(commit)是很容易的; 只要把出错的提交(commit)的名字(reference)做为参数传给命令
```
git revert HEAD

```
这条命令创建了一个撤消了上次提交(HEAD)的新提交
