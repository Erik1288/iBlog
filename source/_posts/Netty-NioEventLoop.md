---
title: Netty-NioEventLoop
date: 2019-02-27 11:06:40
tags:
---


Netty的线程池理念有点像ForkJoinPool，都不是一个线程大池子并发等待一条任务队列，而是每条线程自己一个任务队列，怎么做的？建了N个只有一条线程的线程池。

而且Netty的线程，并不只是简单的阻塞地拉取任务，而是非常辛苦命的在每个循环做三件事情：

先SelectKeys()处理NIO的事件

然后获取2.3里提到的本线程的定时任务，放到本线程的任务队列里

再然后混合2.2里提到的其他线程提交给本线程的任务，一起执行

每个循环里处理NIO事件与其他任务的时间消耗比例，还能通过ioRatio变量来控制，默认是各占50%。

可见，Netty的线程根本没有阻塞等待任务的清闲日子，所以也不使用有锁的BlockingQueue如ArrayBlockingQueue来做任务队列了，而是使用后面提到的无锁的MpscLinkedQueue(Mpsc 是Multiple Producer, Single Consumer的缩写)。