---
title: Disruptor-Internal
date: 2018-08-13 16:46:51
tags:
---

经常会被面试官问到，如果设计一个无锁(Lock-Free)的并发数字自增器，或者无锁的RingBuffer。在开发经验还不够的情况下，确实很难想到改怎么去实现，知道后面接触了CAS后。

常用的，并发情况下访问共享资源，思路总是将并行访问串行化，也就是说，要锁住共享资源。所以，锁还是必要的，而Lock-Free中说的无锁是指没有用到传统的方式对线程进行上锁，也就是sync或者ReentrantLock甚至是wait，不知说真的就没有锁。传统的锁会让线程调度器改变内核级线程的状态，而且有频繁的线程上下文切换，比较耗时，而CAS自旋锁的好处是，线程没有改变running的状态，CAS是CPU级别的操作，不耗时间，属于轻量级锁，坏处也是非常明显的，如果资源上锁时间特别长，阻塞的线程CAS自旋会消耗比较多的CPU资源，而这种CPU消耗，在传统锁上是不存在的，因为线程失去了CPU的执行权力。

自旋锁伪代码：
```
class SpinLock {
    private AtomicBoolean locked = false;
    
    
    public Lock() {
        while(!locked.cas(false->true)){
        }
    }
    
    public Unlock() {
        locked.set(false)
    }
}

```

当然以上代码会有非常大的问题，尤其是Lock()方法被阻塞得太久了，CPU就饿死了(starving)了。所以，当自旋的次数超过了某个阈值后，可以讲自旋锁升级为mutex。



### Disruptor:为什么会这么快
https://ifeve.com/disruptor/
1. 无锁CAS抢占，防止上下文切换
2. 伪共享内存，独立缓存存放独立cursor变量
3. RingBuffer，减少GC


### Disruptor视频资源
http://edu.51cto.com/course/11789.html
http://study.163.com/course/introduction/1004573050.htm
https://www.bilibili.com/video/av15276598/

