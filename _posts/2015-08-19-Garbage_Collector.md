---
layout: post
title:  "Java 垃圾收集器分类总结"
date:   2015-08-19 12:00:00 +0800
categories: GC
---


## 对Java垃圾收集器进行分类总结

### GC之前
在执行GC之前要判断对象是否可以进行垃圾回收，也就是判断对象是否已经"死去"，比较出名的两种判断是`引用计数法`和`可达性分析`。然而前者存在循环引用问题所以JVM中并未采取。
**可达性分析（Reachability Analysis）:**通过一系列的称为"GC Root"的对象作为起点，从这些对象开始搜索，所经过的路径叫做“引用链”，当对象到GC Root没有任何引用链相连则称此对象已经不可用。
可作为**GC Root**的对象包括如下几种：
1. 虚拟机栈中引用的对象
2. 方法区中类静态属性引用的对象
3. 方法区中常量引用的变量
4. 本地方法栈中JNI(Native方法)引用的对象

### 垃圾收集算法
**标记-清除(Mark-Sweep)算法:**标记所有要回收的对象，在标记完成后统一回收所有*被标记*对象。<br>
**复制(Copying)算法：**分两块，将还存活的对象复制到另一块区域，然后将已使用的一次性清除。（回收新生代，一块较大的Eden，两块较小的Survivor 8:1）<br>
**标记-压缩(Mark-Compare)算法：**标记阶段和标记-清理一样，后面并不直接清理而是把存活对象一起移到一端，然后把边界外的一举清除(老年代)。<br>


### 垃圾收集器
待完善