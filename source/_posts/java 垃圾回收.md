---
title: java 垃圾回收
date: 2017-07-01 13:39:23
categories: 
	- java
tags:
	- java
	- 垃圾回收
	- 读书笔记
	- jvm
---

### 分析对象是否已经死亡
* 引用计数法
  优点：实现简单，在远古的jdk版本中是该种方式，python2.7里面的回收也是这种方式
  缺点：容易内存泄露，对象循环依赖，类似于死锁
  
* 可达性分析（一般第一次标记，第二次如果确实没有引用会回收）

### 回收算法
* 标记-清除算法
> 可达性分析后，进行标记，然后清除  
> 缺点：效率不是很高，标记清除后会产生不连续的内存碎片，如果创建大对象时，可分配的连续内存不足时，会提前进行垃圾回收动作。
  
* 复制算法
> 可达性分析后，进行标记，将活着的对象复制到内存的另一块上面，然后将已使用过的内存一次性清除干净。  
> 优点: 实现简单，运行高效，现在的商用虚拟机一般都用来回收新生代，新生代里面分为eden区和2个survivor,比率一般是8:1，回收时将eden区和一个survivor存活的对象复制到另一个空着的survivor区，
> 如果空间不够，会向老年代申请空间，采用分配担保的方式。   
> 缺点：可运行内存缩小为原先的一半，另一半要一直空着，等待下次回收时使用。
* 标记整理算法
> 在老年代，将存活的对象，移向一端，然后清理掉边界外的内存。
* 分代算法
> 将虚拟机根据对象的存活周期不同将内存划分为几块。一般分为新生代和老年代。在不同的年代，采用不同的算法。   

### 可达性分析
* 全局性的引用（如常量和静态属性）
* 执行的上下文（栈帧的本地变量表）

### 垃圾收集器




