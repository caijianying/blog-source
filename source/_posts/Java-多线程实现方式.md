---
title: Java 多线程实现方式
date: 2016-08-21 20:46:59
tags: 编程
categories:
- java
---
# Java 多线程实现方式
## 新建线程
### 1.继承Thread，复写run
实现一个线程最简单的方法
```
new Thread() {
	public void run() {
             System.out.println("I`am a thread!");               
	};
}.start();
```
### 2.实现runnable,实现run
runnable是一个接口类，使用的话，需要实现。
```
new Thread(new Runnable() {
	public void run() {
		// TODO Auto-generated method stub
		System.out.println("I`am a second thread!");     
	}
}).start();
```
### 3.利用jdk中Executor框架实现
Executors类，提供了一系列工厂方法用于创先线程池，返回的线程池都实现了ExecutorService接口。其中包括线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable等。
```
ExecutorService executorService = Executors.newSingleThreadExecutor();
executorService.submit(new Runnable() {
	public void run() {
		System.out.println("I`am a thrid thread!");
	}
});
```

## 带返回值的多线程
结合Future，Callable一起使用，可以取得多线程执行的返回值
### 1.第一种
```
ExecutorService executorService = Executors.newFixedThreadPool(10);
List<Future<String>> results = new ArrayList<Future<String>>();
	for( int i=0;i<10000;i++){
		final int j = i;
		Future<String> result = executorService.submit(new Callable<String>() {
			public String call() throws Exception {
				// TODO Auto-generated method stub
				return "value"+j;
			}
		});
		results.add(result);
	}
for(Future<String>result :results){
	System.out.println(result.get());//当线程没有执行完毕会阻塞
}
executorService.shutdown();
```
### 2.第二种
```
ExecutorService executorService = Executors.newFixedThreadPool(10);
List<Callable<String>> tasks = new ArrayList<Callable<String>>();
for (int i = 0; i < 10000; i++) {
	final int j = i;
	tasks.add(new Callable<String>() {
		public String call() throws Exception {
			// TODO Auto-generated method stub
			return "value" + j;
		}
	});
}
//此时还未执行
List<Future<String>> results = executorService.invokeAll(tasks);//未执行完成阻塞
for (Future<String> result : results) {
	System.out.println(result.get());//当线程没有执行完毕会阻塞
}
executorService.shutdown();
```