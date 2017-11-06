---
title: JVM-Lock
date: 2017-11-06 10:16:02
tags: JVM
---

转载自：http://luojinping.com/2015/07/09/java%E9%94%81%E4%BC%98%E5%8C%96/

### 同步的原理

JVM规范规定JVM基于进入和退出Monitor对象来实现方法同步和代码块同步，但两者的实现细节不一样。代码块同步是使用monitorenter和monitorexit指令实现，而方法同步是使用另外一种方式实现的，细节在JVM规范里并没有详细说明，但是方法的同步同样可以使用这两个指令来实现。monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处， JVM要保证每个monitorenter必须有对应的monitorexit与之配对。任何对象都有一个 monitor 与之关联，当且一个monitor 被持有后，它将处于锁定状态。线程执行到 monitorenter 指令时，将会尝试获取对象所对应的 monitor 的所有权，即尝试获得对象的锁。

### Java对象头

锁存在Java对象头里。如果对象是数组类型，则虚拟机用3个Word（字宽）存储对象头，如果对象是非数组类型，则用2字宽存储对象头。在32位虚拟机中，一字宽等于四字节，即32bit。

| 长度 | 内容 | 说明 |
| --------- |:-----------:|:-----------|
| 32/64bit | Mark Word | 存储对象的hashCode或锁信息等
| 32/64bit | Class Metadata Address | 存储到对象类型数据的指针
| 32/64bit | Array length | 数组的长度（如果当前对象是数组）

Java对象头里的Mark Word里默认存储对象的HashCode，分代年龄和锁标记位。32位JVM的Mark Word的默认存储结构如下：

25 bit	4bit	1bit
是否是偏向锁	2bit
锁标志位
无锁状态	对象的hashCode	对象分代年龄	0	01

在运行期间Mark Word里存储的数据会随着锁标志位的变化而变化。Mark Word可能变化为存储以下4种数据：