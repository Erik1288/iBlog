---
title: Go-Memory-Allocate-And-Collect
date: 2018-08-04 17:20:02
tags:
---

### 写得太好了
https://www.jianshu.com/p/34984105175c


GO语言内存管理子系统主要由两部分组成：内存分配器和垃圾回收器（gc）

内存分配器主要解决小对象的分配管理和多线程的内存分配问题。什么是小对象呢？小于等于32k的对象就是小对象，其它都是大对象。小对象的内存分配是通过一级一级的缓存来实现的，目的就是为了提升内存分配释放的速度以及避免内存碎片等问题。

为什么大部分编程语言都会给对象分配的内存设计「堆Heap」和「栈Stack」这么两块区域？

golang在「堆Heap」和「栈Stack」内存分配上有没有做什么优化？
参考官方（https://groups.google.com/forum/#!msg/golang-nuts/KJiyv2mV2pU/wdBUH1mHCAAJ）
The Go compiler uses escape analysis to find objects whose lifetime is known at compile time, and allocates them on the stack rather than in garbage collected memory. 
So in general, in Go, compared to other languages, a larger percentage of the quickly-unused values that a generational GC looks for are never allocated in GC memory in the first place.  So a generational GC would likely bring less advantage to Go than it does for other 
languages. 


### 简书 tcmalloc
https://www.jianshu.com/p/ec585064a6e1

### 对比JVM中的逃逸分析
http://www.importnew.com/23150.html

### golang的逃逸分析