---
title: linux性能优化笔记之CPU篇
date: 2019-03-02 22:25:36
tags: 笔记
categories:
- linux
---
# linux性能优化笔记之CPU篇

cpu优化用到的命令

- mpstat
进程相关统计工具，cpu/io可以同时对比
- vmstat
内存分析工具
- pidstat
进程分析工具
- perf 使用perf record -g -p < pid>和perf report就足够了


## CPU篇

### 根据指标找工具

![image](https://github.com/TrumanDu/pic_repository/blob/master/%E6%A0%B9%E6%8D%AE%E6%8C%87%E6%A0%87%E6%89%BE%E5%B7%A5%E5%85%B7(CPU%E6%80%A7%E8%83%BD).png?raw=true)

### 根据工具查指标
![image](https://github.com/TrumanDu/pic_repository/blob/master/%E6%A0%B9%E6%8D%AE%E5%B7%A5%E5%85%B7%E6%9F%A5%E6%8C%87%E6%A0%87.png?raw=true)

### 指标工具关联
![image](https://github.com/TrumanDu/pic_repository/blob/master/%E6%8C%87%E6%A0%87%E5%B7%A5%E5%85%B7%E5%85%B3%E8%81%94.png?raw=true)

### 相关命令
查看cpu核数：

```
$ grep 'model name' /proc/cpuinfo | wc -l
```
mpstat 查看 CPU 使用率的变化情况

```
# -P ALL 表示监控所有 CPU，后面数字 5 表示间隔 5 秒后输出一组数据
$ mpstat -P ALL 5
```
pidstat查看具体进程cpu使用率

```
# 间隔 5 秒后输出一组数据

$ pidstat -u 5 1
# 查看进程IO读写情况
$ pidstat -d 
# w可以查看上下文切换情况(进程)  cswch/s 自愿切换（资源不够） nvcswch/s被动切换（时间片到期等中断）
# wt是可以查看到线程
$ pidstat -wt 5
```
vmstat内存分析工具，可以分析上下文切换情况

```
# in中断 sc上下文切换
$ vmstat 1
```

perl系能分析工具，可以追查引起性能问题的函数

```
$ sudo perf top -g -p 30377
```
### 名词解释

1. 平均负载
>简单来说，平均负载是指单位时间内，系统处于**可运行状态**和**不可中断状态**的平均进程数，也就是**平均活跃进程数**，它和 CPU 使用率并没有直接关系。可运行状态顾名思义只得是正在运行的任务，不可中断状态例如等待cpu,等待IO。

2. cpu上下文
>CPU寄存器和程序计数器为任务运行必备依赖环境，即为cpu上下文

>cpu上下文切换包含哪些？
>(1)进程上下文切换(2)线程上下文切换(3)中断上下文切换

### 知识点

1. cpu使用率和平均负载一致吗？

不一定，当平均负载高的时候，任务主要是等待IO型，IO密集型任务，CPU使用率就不一定高。


### 注意

1. pidstat中缺少%wait

centos中版本较低是，安装新版本即有
2. pidstat中%wait与top wa区别

pidstat 中， %wait 表示进程等待 CPU 的时间百分。

top 中 ，iowait%(简写wa) 则表示等待 I/O 的 CPU 时间百分。
