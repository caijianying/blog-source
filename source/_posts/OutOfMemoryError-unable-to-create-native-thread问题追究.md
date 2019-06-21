---
title: OutOfMemoryError-unable to create native thread问题追究
date: 2018-07-22 18:50:10
tags: 问题答疑
categories:
         - linux
         - java
---
# OutOfMemoryError-unable to create native thread问题追究

## 1.问题背景
某天，发现线上elasticsearch 集群，有个节点down了，重启后，过了一个多小时，又down,这下才去认真的查看log,发现在down掉之前，有报以下错误：
```
[67329.555s][warning][os,thread] Failed to start thread - pthread_create failed (EAGAIN) for attributes: stacksize: 1024k, guardsize: 0k, detached.

[67329.557s][warning][os,thread] Failed to start thread - pthread_create failed (EAGAIN) for attributes: stacksize: 1024k, guardsize: 0k, detached.

[2018-07-12T02:47:53,549][ERROR][o.e.b.ElasticsearchUncaughtExceptionHandler] [e11redis28.mercury.corp] fatal error in thread [elasticsearch[e11redis28.mercury.corp][refresh][T#2]], exiting

java.lang.OutOfMemoryError: unable to create native thread: possibly out of memory or process/resource limits reached

t java.lang.Thread.start0(Native Method) ~[?:?]

t java.lang.Thread.start(Thread.java:813) ~[?:?]

t java.util.concurrent.ThreadPoolExecutor.addWorker(ThreadPoolExecutor.java:944) ~[?:?]

t java.util.concurrent.ThreadPoolExecutor.processWorkerExit(ThreadPoolExecutor.java:1012) ~[?:?]

t java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) ~[?:?]

t java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635) ~[?:?]

t java.lang.Thread.run(Thread.java:844) [?:?]
```
困扰了好几天，修改各种jvm 配置发现无用，就在github 上提了个[issues](https://github.com/elastic/elasticsearch/issues/31982),里面详细的描述了该问题及环境的配置。
## 2.该问题产生可能原因
经过查阅大量资料，才发现该错误不是我们一眼看的那么简单。

OutOfMemoryError: Unable to Create New Native Thread

这个错误其实已经告诉我们，该问题可能是两个原因造成的：

- 内存不足

  该内存不足，其实不是指堆内存，而是创建栈内存不足。所以说出现该问题修改Xmx/Xms是无法解决的。默认情况下jvm 创建1个线程，会分配1m空间。这里所指内存不足是机器中可用内存不足。对于该原因的解决办法是给jvm 留够需要的内存，方法有多个：
  >1. 减少Xss配置，这样话可以利用有限的内存创建更多的线程，但是需注意避免值过小栈溢出。
  >2. 减少堆配置，主要还是留下足够内存供jvm 创建线程所用。
  >3. 杀掉其他程序，留出空闲内存。
  
- 机器线程数受限制

  众所周知，linux系统中对进程的创建是有限制的，不可能无限创建的，影响该数量主要是由以下三个方面：
  >1. /proc/sys/kernel/threads-max 
  >2. max_user_process（ulimit –u）
  >3. /proc/sys/kernel/pid_max
  
  通过这**三个参数共同**决定了linux机器中能创建线程数量
## 3.linux 进程相关信息
在这里主要是想写一些命令帮助人们分析linux进程下的线程等信息

- 查看linux所有用户下所有进程数

   ```
   ps -eLo ruser|awk 'NR>1'|sort|uniq -c
   ```
- 查看进程中的线程数

  ``` 
  ps –o nlwp 27989 
  ```
- 查看某个进程下的线程

  ``` 
   top -H -p <pid>
  ps -eLf | grep <pid>
  ```
## 4.检测系统参数
1. 查看内存 free -g
2. 查看limit

   - ulimit -a
   - cat /proc/sys/kernel/threads-max 
   - cat /proc/sys/kernel/pid_max


## 5.问题模拟验证
经过以上步骤分析，写了个脚本，监控该机器上内存使用情况及线程使用情况，发现问题出现在有个程序会创建大量的线程，该程序是java 程序。查看机器pid_max为32768，ulimit -u  未限制大小。 总结一下这个机器线程最多允许创建 32768。


以下是验证代码：注意该代码有一定危险性，请勿在window 下运行。


```
int i =0;
while(true) {
	Thread thread = new Thread(new Runnable() {
		@Override
		public void run() {
			while(true) {
				try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	});
	
	thread.start();
	i++;
	System.out.println("curent thread num:"+i);
}
```

运行该代码，当线程增长到3.1w时，报该问题。

## 6.本次问题原因
本次问题是发现是有个程序创建了大量线程，造成达到linux 机器允许最大的线程数。切记这个不是一个参数控制的，多个参数控制pid_max，一般人可能会忽略！

## 7.问题解决办法
根据不同问题选择不同办法，详见第2点 [该问题产生可能原因](#2.该问题产生可能原因)
## 8.参考

1. [We are out of memory](https://www.elastic.co/blog/we-are-out-of-memory-systemd-process-limits)
2. [为何线程有PID](https://blog.csdn.net/lh2016rocky/article/details/55671656)
3. [Troubleshoot OutOfMemoryError: Unable to Create New Native Thread](https://dzone.com/articles/troubleshoot-outofmemoryerror-unable-to-create-new)