---
title: SourceCode-AbstractQueuedSynchronizer
date: 2017-12-03 11:15:58
tags: 源码分析
---



AQS使用一个int类型的成员变量state来表示同步状态，当`state > 0`时表示已经获取了锁，当`state = 0`时表示释放了锁。它提供了三个方法（getState()、setState(int newState)、compareAndSetState(int expect,int update)）来对同步状态state进行操作，当然AQS可以确保对state的操作是安全的。
AQS通过内置的FIFO同步队列来完成资源获取线程的排队工作，如果当前线程获取同步状态失败（锁）时，AQS则会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。

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





### 很好的文章
http://blog.csdn.net/hengyunabc/article/details/28126139

park和unpark的灵活之处
上面已经提到，unpark函数可以先于park调用，这个正是它们的灵活之处。

一个线程它有可能在别的线程unPark之前，或者之后，或者同时调用了park，那么因为park的特性，它可以不用担心自己的park的时序问题，否则，如果park必须要在unpark之前，那么给编程带来很大的麻烦！！

考虑一下，两个线程同步，要如何处理？

在Java5里是用wait/notify/notifyAll来同步的。wait/notify机制有个很蛋疼的地方是，比如线程B要用notify通知线程A，那么线程B要确保线程A已经在wait调用上等待了，否则线程A可能永远都在等待。编程的时候就会很蛋疼。

另外，是调用notify，还是notifyAll？

notify只会唤醒一个线程，如果错误地有两个线程在同一个对象上wait等待，那么又悲剧了。为了安全起见，貌似只能调用notifyAll了。

park/unpark模型真正解耦了线程之间的同步，线程之间不再需要一个Object或者其它变量来存储状态，不再需要关心对方的状态。

HotSpot里park/unpark的实现

每个java线程都有一个Parker实例，Parker类是这样定义的：

``` C++ 
class Parker : public os::PlatformParker {  
private:  
  volatile int _counter ;  
  ...  
public:  
  void park(bool isAbsolute, jlong time);  
  void unpark();  
  ...  
}  
class PlatformParker : public CHeapObj<mtInternal> {  
  protected:  
    pthread_mutex_t _mutex [1] ;  
    pthread_cond_t  _cond  [1] ;  
    ...  
} 
```