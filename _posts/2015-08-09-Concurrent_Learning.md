---
layout: post
title:  "Java 并发学习"
date:   2015-08-09 12:00:00 +0800
categories: Concurrent 
---


### 启动
启动一个新线程来执行并发有多种方法，最基本的有如下几种：<br>
1. 定义任务，让其实现Runnable接口，并编写run()方法。然后把其传给Thread构造器有Thread的start()方法为线程执行必要的初始化操作。
2. 定义类继承Thread类，重写run()方法，并调用其start()方法启动线程。
3. 定义任务，让其实现callable接口，在任务完成时将返回一个具有类型参数的返回值。必须使用ExecutorService.submit()方法调用。
4. 单个Executor被用来创建和管理系统中所有的任务。
	- CachedThreadPool:线程执行时创建于所需数量相同的线程，在回收旧线程时停止创建新线程
	- FixedThreadPool:一次性预先执行代价高昂的线程分配，因此可以限制线程数量。
	- SingleThreadPool:像线程数为1的FixedThreadPool,向它提交多个任务这些任务将排队。

![executor接口体系](/images/posts/Concurrent/executor_interface.png "executor接口体系")
<center>Executor接口体系类图</center>

<br/>


### 几种常见操作

1. 休眠：使任务终止执行给定的时间。`TimeUnit.MILLISECONDS.sleep(time)`
2. 优先级：将该任务的重要性传递给调度器。`get & setPriority()`
3. 让步:让出CPU给其他线程运行`yield()`。
4. 后台线程：当所有非后台程序结束时，程序也就终止，同时会杀死进程中所有后台进程。`setDaemon(true)` 必须在线程调用之前调用。
5. 加入一个线程(join)：如果某个线程在另一个线程t上调用 `t.join()`,此线程将被挂起，直到目标线程t结束才恢复。
<br/>还可以在`join(time)`带上超时参数，超过时间join()方法总能返回。
6. 不能捕获从线程中逃逸的**异常**，只能在本线程中处理。但是可以在Thread对象上附着一个异常处理器：`Thread.UncaughtExceptionHandler.uncaughtException()`来处理。 

### 共享受限资源(互斥)
