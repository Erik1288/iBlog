---
title: RocketMQ——高性能秘籍之刷盘机制
date: 2017-09-19 18:47:17
tags: RocketMQ
---

### 逻辑Offset队列: ConsumerQueue


### 物理Offset队列: CommitLog

### MappedByteBuffer
操纵MappedByteBuffer的线程或者进程必须对某一个文件的映射Buffer有独占权，
在设计上，消息的顺序是由CommitLog决定，所以CommitLog在Append新的消息时，必须上锁进行互斥。

*传统的synchronized叫做monitor lock，当一个线程进入了synchronized的代码块时，我们说，该线程own（拥有）了monitor lock。这种锁是一种重量级锁，用mutual exclusive（互斥）的特性来实现了同步的需求。
*自旋锁，JDK1.6引进，我们知道，线程状态与状态的切换，是需要内核参与的，简单点来讲，这个过程是需要点时间的。线程B已经own了一个锁，这是线程A去尝试获取锁，本来线程A应该要挂起，JVM不让它挂起，让A在那里做自旋操作，JVM要赌当前持有锁的B会很快释放锁。如果线程B确实很快释放了锁，那对于A来讲是一个非常好事情，因为A可以不用切换状态，立刻持有锁。那什么时候会用到呢？http://blog.csdn.net/u013080921/article/details/42676231

#### Spin Lock(自旋锁)

#### ReentrantLock(重入锁)

