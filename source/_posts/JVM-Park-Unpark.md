---
title: JVM-Park-Unpark
date: 2018-08-15 09:52:45
tags: JVM
---

Java并发的工具非常多，但是底层都是基于sun.misc.Unsafe#park和sun.misc.Unsafe#unpark这两个native，本文就来研究下究竟park和unpark表达的语义是什么。

首先，看Java doc
```
    // LockSupport class
    /**
     * Disables the current thread for thread scheduling purposes unless the
     * permit is available.
     *
     * <p>If the permit is available then it is consumed and the call
     * returns immediately; otherwise the current thread becomes disabled
     * for thread scheduling purposes and lies dormant until one of three
     * things happens:
     *
     * <ul>
     *
     * <li>Some other thread invokes {@link #unpark unpark} with the
     * current thread as the target; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread; or
     *
     * <li>The call spuriously (that is, for no reason) returns.
     * </ul>
     *
     * <p>This method does <em>not</em> report which of these caused the
     * method to return. Callers should re-check the conditions which caused
     * the thread to park in the first place. Callers may also determine,
     * for example, the interrupt status of the thread upon return.
     */
    public static void park() {
        UNSAFE.park(false, 0L);
    }
```
park方法没有参数，结合注释，想表达的意思是，线程A自己调用park()，将禁止线程调度器对A自己的调度，当然方法自然是阻塞住的。
而要让线程调度器重新调度A，需要任意一个以下事情发生：
1. 另外一个线程B调用unpark(A)时，并且参数是阻塞的那个A线程，线程A将被重新调度
2. 另外一个线程B调用interrupt(A)时
3. 不合逻辑的调用

重要的是第一条



### park()和unpark在jvm层是怎么实现的？
参考
https://blog.csdn.net/hengyunabc/article/details/28126139