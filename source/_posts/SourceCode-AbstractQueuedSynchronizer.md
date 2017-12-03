---
title: SourceCode-AbstractQueuedSynchronizer
date: 2017-12-03 11:15:58
tags: 源码分析
---


http://guochenglai.com/2016/06/06/java-concurrent5-aqs-code-analysis/

### 基于AQS的公平与非公平锁设计

### fair

严格的先来先到，如果有人来排队，不能插队，一定要排到队尾！！

### nonfair

总体是先来先到，但新来的可以在队头进行插队

``` java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
         acquire(1);
 }
```

``` java
/**
 * Releases in exclusive mode.  Implemented by unblocking one or
 * more threads if {@link #tryRelease} returns true.
 * This method can be used to implement method {@link Lock#unlock}.
 *
 * @param arg the release argument.  This value is conveyed to
 *        {@link #tryRelease} but is otherwise uninterpreted and
 *        can represent anything you like.
 * @return the value returned from {@link #tryRelease}
 */
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```


### 基于AQS的倒计时锁设计

### 基于AQS的semaphore设计