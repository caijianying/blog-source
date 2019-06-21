---
title: 如何精准化爬取github repository给star用户信息
date: 2019-03-13 20:41:26
tags: 爬虫
categories:
- python
---
## 前言
最近开源了一个redis内存分析一站式平台RCT,想让更多的人指定我们这个项目，正好看到同类几个开源项目，就想着爬取关注的用户，给他们发邮件，推广一下我们的项目。
## 实践
因为本地linux机器安装的python版本为2.7.0，因此使用python2语法
### 安装依赖
```
$ pip install PyGithub
```
### 代码实现
```
from github import Github
import os
import codecs
def create(newfile):
    if not os.path.exists(newfile):
         f = open(newfile,'w')
         print newfile
         f.close()
    return
fileName='rct.txt'
respository='xaecbd/RCT'
g=Github("token",timeout=300)

repo=g.get_repo(respository)
stargazers=repo.get_stargazers_with_dates()
create(fileName)
with codecs.open(fileName,'a', 'utf-8') as f:
    for people in stargazers:
        print people.user.email
        if people.user.email is not None:
            if people.user.name is None:
                name = people.user.email
            else:
                name = people.user.name
            data=(name)+":"+(people.user.email)+"\r\n"
            f.write(data)
```
### 运行
```
python crawer.py
```
### 总结
在运行过程中发现关于用户数据是输出的时候才再去github查询的，这也就意味着，及时stargazers已经获取，还会继续去调用github api。可能会存在网络问题。
## 参考
1. [怎样通过GitHub API获取某个项目被Star的详细信息](https://blog.csdn.net/qysh123/article/details/79782963)
2. [PyGithub](https://github.com/PyGithub/PyGithub)