---
title: linux运维问题
date: 2016-07-18 21:16:01
tags: 运维
categories:
- linux
---
# linux 运维
## 1.后台执行任务
问题描述：

在linux中执行一些服务，我们通常会使用sudo命令，以root权限运行，再以nohup后台运行，但是有的情况下需要交互操作。

解决方案：

1.首先输入命令，
```
sudo nohup commandline
```
2.使用以下快捷键中断
```
ctrl+z
```
3.在命令台中输入
```
bg
```
## 2.监控linux磁盘IO
可以使用**iostat**命令,iostat还有一个比较常用的选项-x，该选项将用于显示和io相关的扩展数据。（可以结合grep一起使用便于过滤具体磁盘）

**命令解释：**
参数 -d 表示，显示设备（磁盘）使用状态；-k某些使用block为单位的列强制使用Kilobytes为单位(同样可以用m表示兆)；2表示，数据显示每隔2秒刷新一次。
```
iostat -d -k 2
```
输出信息解释
```
tps：该设备每秒的传输次数（Indicate the number of transfers per second that were issued to the device.）。"一次传输"意思是"一次I/O请求"。多个逻辑请求可能会被合并为"一次I/O请求"。"一次传输"请求的大小是未知的。

kB_read/s：每秒从设备（drive expressed）读取的数据量；
kB_wrtn/s：每秒向设备（drive expressed）写入的数据量；
kB_read：读取的总数据量；
kB_wrtn：写入的总数量数据量；这些单位都为Kilobytes。
```
