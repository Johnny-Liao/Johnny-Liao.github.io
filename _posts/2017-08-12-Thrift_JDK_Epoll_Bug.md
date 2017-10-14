---
layout: post
title:  "当 Thrift 遇到 JDK Epoll Bug"
date:   2017-08-12 12:00:00 +0800
categories: Thrift JDK Epoll 
---

### 前言
相信不少 Java server 端程序深受臭名昭著的 JDK Epoll Bug 其害。一旦触发所有 Selector 线程处于空转状态不能自拔，直至 cpu 跑满，不再处理外来连接。对于外部调用者来说这意味着服务不可用，这真是一场灾难。

笔者不才，前段时间在使用 Thrift 作为 server 端 rpc 框架对外提供服务时刚好触发了此 bug ，在此记录定位问题的过程和解决办法，供各位参考，望再有后来人碰到此问题不在受此困扰。话不多说，先来复盘。


###  问题定位

话说服务端程序都已开心码完，在做最后的压测时，突然发现压测程序请求不到服务直至访问超时。赶紧到所在服务器查看服务状况，结果发现多个 cpu 跑满了，如下图：

![cpu run full](https://static.oschina.net/uploads/img/201708/12152533_v2Oc.png "cpu run full")

于是赶紧用 `top -Hp $PID` 查看下所在进程的线程情况，结果如下图：

![thread info in progress](https://static.oschina.net/uploads/img/201708/12153501_UUMW.png "thread info in progress")

按 cpu 使用量排序后，发现前面几个线程 cpu 使用量接近 100% 。 Java 程序快用 jstack 查看这个线程在干嘛吧，定位到 pid=177 ( 0xb1 ) 的线程堆栈信息如下：

![thread details](https://static.oschina.net/uploads/img/201708/12153937_rWUy.png "thread details")

从堆栈信息可以看到触发点位于 Thrift 的 TThreadedSelectorServer.select() 方法中，查看源码可知其不过是调用了JDK Selector.select() 方法。（注：为描述方便笔者把实例方法调用使用类方法调用形式展示）

![TThreadedSelectorServer.select()](https://static.oschina.net/uploads/img/201708/12155718_pFCR.png "TThreadedSelectorServer.select()")

看现象初步认为是压测时连接数过多导致 Selector 线程一直处于繁忙状态，可后来发现停掉压测 cpu 使用量还是居高不下。

笔者曾一度陷入困境，后经高人同事提点道：会不会是JDK Epoll Bug ？我一想这个 bug 官方不是修复了吗，后来查证发现：官方声称在JDK1.6版本的update18修复了该问题，然而只是降低了发生的概率而已，它并没有被根本解决。该 bug 以及相关的问题如下：

[http://bugs.java.com/bugdatabase/view_bug.do?bug_id=6403933](http://bugs.java.com/bugdatabase/view_bug.do?bug_id=6403933)

[http://bugs.java.com/bugdatabase/view_bug.do?bug_id=2147719](http://bugs.java.com/bugdatabase/view_bug.do?bug_id=2147719)

笔者当时环境为：

![work JRE](https://static.oschina.net/uploads/img/201708/12161637_w08W.png "work JRE")

看来到 jdk 1.8 这个问题也没解决，为 Oracle 感到小小的尴尬啊。

### 解决办法

JDK 的 bug 让使用者极其的尴尬啊，因为我们又不能更改其源码。好在好的搜索引擎拉低了学习的门槛，在茫茫互联网中帮我找到了一位高人总结的解决办法，其中有详细写明触发原因及解决办法，详见：

[应用服务器中对JDK的epoll空转bug的处理](http://www.10tiao.com/html/308/201602/401718035/1.html)

笔者简述下解决办法：
1. 确认程序触发 epoll bug ：在一定时间间隔内触发了设定阈值次数的空转，则认定触发 epoll bug
2. 修复：通过重建 Selector 方式，新 Selector 注册问题 Selector 所有事件后替换它。

定位到问题及知道解决办法后，接下来就赶紧修复它吧。

按照 Thrift 官方 Developers 贡献指南一步步来：
- 先在 Apeach Jira中建一个 issue ： [https://issues.apache.org/jira/browse/THRIFT-4251](https://issues.apache.org/jira/browse/THRIFT-4251)
- 再把修复好 epoll bug 的代码 pull request 到 thrift 官方 git 上：[https://github.com/apache/thrift/pull/1313](https://github.com/apache/thrift/pull/1313)

至此问题修复完成。感兴趣者可以关注上述 issue 来获取官方修复的最新进展。



参考博客：

[应用服务器中对JDK的epoll空转bug的处理](http://www.10tiao.com/html/308/201602/401718035/1.html)

[Java NIO通信框架在电信领域的实践](http://www.infoq.com/cn/articles/practice-of-java-nio-communication-framework)

转载此博客请注明来源：[https://my.oschina.net/johnnyliao/blog/1507141](https://my.oschina.net/johnnyliao/blog/1507141)

如上为笔者开源中国发表的文章，如有疑问可以留言。

<p>
注：直至09月23日，Thrift 官方已经合并笔者代码，并修复此问题。
</p>
