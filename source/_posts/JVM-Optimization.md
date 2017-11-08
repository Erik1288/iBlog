---
title: JVM-Optimization
date: 2017-11-06 14:43:26
tags: JVM
---

http://blog.csdn.net/matt8/article/details/52298397

Java ReentrantLock（重入锁）带来的改变


### 前言
> ReentrantLock称为重入锁，它比内部锁synchronized拥有更强大的功能，它可中断、可定时，JDK5中，在高并发的情况下，它比synchronized有明显的性能优势，在JDK6中由于jvm的优化，两者差别不是很大。


synchronized原语和ReentrantLock在一般情况下没有什么区别，但是在非常复杂的同步应用中，请考虑使用ReentrantLock，特别是遇到下面2种需求的时候。 
1.某个线程在等待一个锁的控制权的这段时间需要中断 
2.需要分开处理一些wait-notify，ReentrantLock里面的Condition应用，能够控制notify哪个线程 
3.具有公平锁功能，每个到来的线程都将排队等候 

先说第一种情况，ReentrantLock的lock机制有2种，忽略中断锁和响应中断锁，这给我们带来了很大的灵活性。比如：如果A、B2个线程去竞争锁，A线程得到了锁，B线程等待，但是A线程这个时候实在有太多事情要处理，就是一直不返回，B线程可能就会等不及了，想中断自己，不再等待这个锁了，转而处理其他事情。这个时候ReentrantLock就提供了2种机制，第一，B线程中断自己（或者别的线程中断它），但是ReentrantLock不去响应，继续让B线程等待，你再怎么中断，我全当耳边风（synchronized原语就是如此）；第二，B线程中断自己（或者别的线程中断它），ReentrantLock处理了这个中断，并且不再等待这个锁的到来，完全放弃。（如果你没有了解java的中断机制，请参考下相关资料，再回头看这篇文章，80%的人根本没有真正理解什么是java的中断，呵呵） 

这里来做个试验，首先搞一个Buffer类，它有读操作和写操作，为了不读到脏数据，写和读都需要加锁，我们先用synchronized原语来加锁，如下： 

``` java
package com.eric.lock;

public class Buffer {

    private Object lock;

    public Buffer() {
        lock = this;
    }

    public void write() {
        synchronized (lock) {
            long startTime = System.currentTimeMillis();
            System.out.println("开始往这个buff写入数据…");
            // 模拟要处理很长时间
            for (; ; ) {
                if (System.currentTimeMillis() - startTime > Integer.MAX_VALUE)
                    break;
            }
            System.out.println("终于写完了");
        }
    }

    public void read() {
        synchronized (lock) {
            System.out.println("从这个buff读数据");
        }
    }
}
```

接着，我们来定义2个线程，一个线程去写，一个线程去读。