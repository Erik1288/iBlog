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

#### Spin Lock(自旋锁)

#### ReentrantLock(重入锁)