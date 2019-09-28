---
title: JVM-Valotile
date: 2018-11-20 20:36:21
tags: JVM
---

### 示例代码
https://www.cnblogs.com/chenyangyao/p/5269622.html


Valotile的两层语义
### Memory bar

https://www.jianshu.com/p/ef8de88b1343


### importjava 深度文章
http://www.importnew.com/29860.html
JMM内存模型
在上面描述中可以看到硬件为我们提供了很多的额外指令来保证程序的正确性。但是也带来了复杂性。JMM为了方便我们理解和使用，提供了一些抽象概念的内存屏障。注意，下文开始讨论的内存屏障都是指的是JMM的抽象内存屏障，它并不代表实际的cpu操作指令，而是代表一种效果。

LoadLoad Barriers
该屏障保证了在屏障前的读取操作效果先于屏障后的读取操作效果发生。在各个不同平台上会插入的编译指令不相同，可能的一种做法是插入也被称之为smp_rmb指令，强制处理完成当前的invalidate queue中的内容
StoreStore Barriers
该屏障保证了在屏障前的写操作效果先于屏障后的写操作效果发生。可能的做法是使用smp_wmb指令，而且是使用该指令中，将后续写入数据先写入到store buffer的那种处理方式。因为这种方式消耗比较小
LoadStore Barriers
该屏障保证了屏障前的读操作效果先于屏障后的写操作效果发生。
StoreLoad Barriers
该屏障保证了屏障前的写操作效果先于屏障后的读操作效果发生。该屏障兼具上面三者的功能，是开销最大的一种屏障。可能的做法就是插入一个smp_mb指令来完成。

内存屏障在volatile关键中的使用
内存屏障在很多地方使用，这里主要说下对于volatile关键字，内存屏障的使用方式。

在每个volatile写操作的前面插入一个StoreStore屏障。
在每个volatile写操作的后面插入一个StoreLoad屏障。
在每个volatile读操作的后面插入一个LoadLoad屏障。
在每个volatile读操作的后面插入一个LoadStore屏障。
上面的内存屏障方式主要是规定了在处理器级别的一些重排序要求。而JMM本身，对于volatile变量在编译器级别的重排序也制定了相关的规则。可以用下面的图来表示

https://www.cnblogs.com/yzwall/p/6661528.html


```
private volatile int handlerState = INIT;


/**
     * Makes best possible effort to detect if {@link ChannelHandler#handlerAdded(ChannelHandlerContext)} was called
     * yet. If not return {@code false} and if called or could not detect return {@code true}.
     *
     * If this method returns {@code true} we will not invoke the {@link ChannelHandler} but just forward the event.
     * This is needed as {@link DefaultChannelPipeline} may already put the {@link ChannelHandler} in the linked-list
     * but not called {@link ChannelHandler#handlerAdded(ChannelHandlerContext)}.
     */
    private boolean invokeHandler() {
        // Store in local variable to reduce volatile reads.
        int handlerState = this.handlerState;
        return handlerState == ADD_COMPLETE || (!ordered && handlerState == ADD_PENDING);
    }
```