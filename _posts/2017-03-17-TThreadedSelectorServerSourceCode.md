---
layout: post
title:  "Thrift源码分析--TThreadedSelectorServer线程模型"
date:   2017-03-17 12:00:00 +0800
categories: Thrift TThreadedSelectorServer 网络模型 
---

Thrift的网路服务模型之一的 TThreadedSelectorServer 实现了多Reactor线程模型，请先理解线程模型再深入分析源码。线程模型的讲解此处不再赘述，给出一张总结图如下：

![TThreadedSelectorServer线程模型](https://static.oschina.net/uploads/img/201703/27122406_jasT.png "TThreadedSelectorServer线程模型")

截图来自：[Thrift 服务模型和序列化机制深入学习](http://www.liuqianfei.com/article/065b0f1ee59a4cf0b94a84c4e33af127)

下面从一次完整的服务调用过程来分析源码：

### 1. 服务入口
先来看看服务入口函数 serve() ，TThreadedSelectorServer 的 serve() 方法继承自 AbstractNoblockingServer 抽象类。

![serve()](https://static.oschina.net/uploads/img/201703/27122617_LBkH.png)

其中startThread() 方法为抽象方法，在 TThreadedSelectorServer 中实现如下：

![startThread()](https://static.oschina.net/uploads/img/201703/27122852_IBUo.png)

在 startThread() 方法中启动了一个 AcceptThread（用来接收网络socket）和多个 SelectorThread（进行网络I/O操作）。接下来具体分析这两个线程对象。

### 2. Acceptor
直接看其 run() 方法可知，内部不断的循环监听新来的网络连接：

![AcceptThread.run()](https://static.oschina.net/uploads/img/201703/27123051_TKMe.png)

将就绪的连接分配到对应 SelectorThread 中的 BlockingQueue 队列中：

![doAddAccept](https://static.oschina.net/uploads/img/201703/27123131_VyWm.png)

在 handleAccept() 方法中使用了 SelectorThreadLoadBalancer 负载均衡器轮循分发就绪的连接到对应的 SelectorThread 上去：

![handleAccept()](https://static.oschina.net/uploads/img/201703/27123158_0cdD.png)

![SelectorThreadLoadBalancer](https://static.oschina.net/uploads/img/201703/27123322_Hjzw.png)

接下来看 Selector 如何处理连接。

### 3. Selector

同样线程对象直接看其 run() 方法：可以看到待网络 I/O 事件发生时处理对应的 I/O 事件。

![Selector.run()](https://static.oschina.net/uploads/img/201703/27123432_7PEf.png)

深入对应读写方法可以发现，真正数据的读写操作是调用 FrameBuffer 的 read()/write() 方法，后续我们再对 FrameBuffer 类做讲解，接下来看下工作线程池部分的实现。

#### 4. 任务工作线程池

使用的是Java 提供的 ExecutorService 线程池来管理工作线程。线程池配置来至 Args 配置管理类中：

![工作线程池](https://static.oschina.net/uploads/img/201703/27123506_upZU.png)

默认实现是 FixedThreadPool 大小为 5，可以自定义调节大小和修改线程池实现。

接下来我们根据其调用处来跟进线程池的用出：

##### 使用处一：Acceptor中的 handleAccept 方法用来处理 FAIR_ACCEPT 策略的新连接。

![handleAccept ](https://static.oschina.net/uploads/img/201703/27123639_OCcd.png)

##### 使用处二：当SelectorThread 完成 read 操作后，回调 frameBuffer.invoke() 方法 -- 最后回调业务处理方法。

![handleRead](https://static.oschina.net/uploads/img/201703/27123718_JhC2.png)


从线程池中取出线程去执行 Invocation 线程对象：

![requestInvoke](https://static.oschina.net/uploads/img/201703/27123747_xnXn.png)

Invocation 封装 FrameBuffer 的 invoke() 方法到单独的线程中。

![Invocation](https://static.oschina.net/uploads/img/201703/27123823_iLST.png )

FrameBuffer 中的 invoke() 方法：

![invoke()](https://static.oschina.net/uploads/img/201703/27123850_thNa.png)

从图中可以看到回调 TProcessor 接口的 process(in, out) 方法。
Thrift IDL 生成代码中的 Processor 类实现 TProcessor 接口，Processor 中对应的业务方法实际是调用 Iface 接口定义的业务方法，我们自己的业务代码实现 Iface 接口完成自己的业务处理逻辑。

![TProcessor](https://static.oschina.net/uploads/img/201703/27123916_oA9F.png)

完成业务逻辑处理后，会调用 responseReady() 方法，其内部判断数据读取完成和业务处理完成后，将更改 FrameBufferState 为 write 状态，然后开始回写结果给调用端。

至此完成一次完整的服务调用过程。
