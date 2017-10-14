---
layout: post
title:  "IoC 容器的实现"
date:   2015-08-19 12:00:00 +0800
categories: IoC Spring
---

**IoC(Inverse of Control 控制反转)：**依赖对象的获得被反转了。

基于这个结论为控制反转取了个更好听的名字`依赖注入`。<br/>



Spring的IoC容器可以在对象生成或初始化时直接将数据注入到对象中。


### IoC中容器的设计与实现

- 两个主要的容器系列：
   1. 一个是实现**BeanFactory**的简单容器系列，只实现了容器的最基本功能。
   2. 另一个是实现**ApplicationContext**应用上下文，作为容器的高级形式存在。
  
- Spring IoC容器的设计
	1. 先来看看书中的接口图

		![IoC容器接口设计-已变](/images/posts/IoC/Interface.png "Ioc Interface")

	2. 然后查看Spring-4.2.0.RELEASE中的hierarchy可以发现ApplecationContext已经不再继承AutoWireCapableBeanFactory接口了。

	3. 附图两张分别为BeanFactory继承体系图和ApplicationContext继承接口图

![HierarchyOfBeanFactory](/images/posts/IoC/BeanFactoryHie.png)

![SuperTypeHierarchyOfApplicationContext](/images/posts/IoC/ApplicationContexHie.png)<br/>

<br/>

由于更改太多，一图顶千言，自己直接画张图来表示。如下：

<br/>
![Spring4.0-BeanFactoryHierarchy](/images/posts/IoC/Spring4.0-BeanFactoryHierarchy.png)
<center>Spring-4.2.0.RELEASE中从BeanFactory出发的接口体系。</center>

<br/>
![Spring4.0-BeanFactoryHierarchy-all](/images/posts/IoC/Spring4.0-BeanFactoryHierarchy-all.jpg)
<center>Spring-4.2.0.RELEASE中从BeanFactory出发的接口体系全图。</center>
