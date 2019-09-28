---
title: JVM-FreeList
date: 2018-10-11 19:46:45
tags: JVM
---

https://blog.csdn.net/happyqwz/article/details/8723016
```
方法是重载运算符，重新定义new，和delete方法。英文书，原理细节描述不清，我的大致理解是，new操作的时候，要跑到内存中去取一块内存单元，CPU在跑着程序，如果能从较近的地方取内存肯定要比去远的地方要好的多。于是，就采取自行管理链表所需内存的方式，定义一freeList链表，为Link类的静态成员变量，初始值为NULL，表示没有“近内存”。调用new方法的时候，如果freelist不为空，那么就在freelist中取一块内存（近内存），如果为NULL，那么只能调用原new操作符，调用“远内存”。调用delete方法的时候，释放的内存不丢弃，而是把它加载到freelist中，供程序之后重新使用。这样的话，程序的运行就可以在小范围内存区域中存取，那么注定要比系统定义的new delete方法要快得多吧...
```