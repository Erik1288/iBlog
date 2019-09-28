---
title: JVM-GC-G1-Tuning
date: 2018-12-24 16:27:27
tags: JVM
---


主要参考
http://presentations2015.s3.amazonaws.com/40_presentation.pdf


## G1优化思路
1. 第一部，确定系统类型
吞吐优先，适用于大规模离线计算系统

暂停时间优先，适用于实时计算并反馈的系统
-XX:MaxGCPauseMillis=50

## Safepoint优化
如果连续打印以下日志
```
2018-12-18T15:14:27.423+0800: 88.781: Total time for which application threads were stopped: 0.0022266 seconds, Stopping threads took: 0.0005306 seconds
2018-12-18T15:14:28.425+0800: 89.784: Total time for which application threads were stopped: 0.0024547 seconds, Stopping threads took: 0.0005578 seconds
2018-12-18T15:14:29.428+0800: 90.787: Total time for which application threads were stopped: 0.0027979 seconds, Stopping threads took: 0.0006438 seconds
2018-12-18T15:14:31.431+0800: 92.790: Total time for which application threads were stopped: 0.0028555 seconds, Stopping threads took: 0.0006590 seconds
2018-12-18T15:14:32.434+0800: 93.793: Total time for which application threads were stopped: 0.0029690 seconds, Stopping threads took: 0.0006541 seconds
2018-12-18T15:14:33.437+0800: 94.796: Total time for which application threads were stopped: 0.0031094 seconds, Stopping threads took: 0.0006861 seconds
2018-12-18T15:14:34.441+0800: 95.799: Total time for which application threads were stopped: 0.0033426 seconds, Stopping threads took: 0.0007415 seconds
```
### 查看safepoint
-XX:+PrintSafepointStatistics
-XX:PrintSafepointStatisticsCount=1
### Revoke Bias优化
-XX:-UseBiasedLocking
### 非常多的"no vm operation"优化，即所谓的Guaranteed Safepoint，用于做monitor的清理
-XX:+UnlockDiagnosticVMOptions
-XX:-DisplayVMOutput
-XX:+LogVMOutput
-XX:LogFile=/opt/app/logs/vm.log
-XX:GuaranteedSafepointInterval=300000


```
完整启动参数
set JAVA_OPTS
JAVA_OPTS="-server -Xms17944m -Xmx17944m -Xss256k -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m"

# JAVA_OPT="$JAVA_OPT -XX:MaxDirectMemorySize=30g"

#performance Options
JAVA_OPTS="$JAVA_OPTS -XX:+UseG1GC"

JAVA_OPTS="$JAVA_OPTS -XX:+PrintSafepointStatistics"
JAVA_OPTS="$JAVA_OPTS -XX:PrintSafepointStatisticsCount=1"
JAVA_OPTS="$JAVA_OPTS -XX:+UnlockDiagnosticVMOptions"
JAVA_OPTS="$JAVA_OPTS -XX:-DisplayVMOutput"
JAVA_OPTS="$JAVA_OPTS -XX:+LogVMOutput"
JAVA_OPTS="$JAVA_OPTS -XX:LogFile=/opt/logs/cobar/vm.log"
JAVA_OPTS="$JAVA_OPTS -XX:GuaranteedSafepointInterval=300000"

JAVA_OPTS="$JAVA_OPTS -XX:MaxGCPauseMillis=50"
JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=16"
JAVA_OPTS="$JAVA_OPTS -XX:ConcGCThreads=4"
JAVA_OPTS="$JAVA_OPTS -XX:SoftRefLRUPolicyMSPerMB=0"
JAVA_OPTS="$JAVA_OPTS -XX:+ParallelRefProcEnabled"
JAVA_OPTS="$JAVA_OPTS -XX:+DisableExplicitGC"
JAVA_OPTS="$JAVA_OPTS -verbose:gc"
JAVA_OPTS="$JAVA_OPTS -Xloggc:/opt/logs/cobar/%p_gc.log"
JAVA_OPTS="$JAVA_OPTS -XX:+AggressiveOpts"
JAVA_OPTS="$JAVA_OPTS -XX:-UseBiasedLocking"
JAVA_OPTS="$JAVA_OPTS -XX:+UseFastAccessorMethods"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGCDetails"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGCDateStamps"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGCApplicationStoppedTime"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintAdaptiveSizePolicy"
JAVA_OPTS="$JAVA_OPTS -XX:+UseGCLogFileRotation"
JAVA_OPTS="$JAVA_OPTS -XX:NumberOfGCLogFiles=5"
JAVA_OPTS="$JAVA_OPTS -XX:GCLogFileSize=30m"
JAVA_OPTS="$JAVA_OPTS -XX:-OmitStackTraceInFastThrow"
JAVA_OPTS="$JAVA_OPTS -XX:+AlwaysPreTouch"
JAVA_OPTS="$JAVA_OPTS -XX:MaxTenuringThreshold=15"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintHeapAtGC"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintFlagsFinal"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintPromotionFailure"
JAVA_OPTS="$JAVA_OPTS -XX:+HeapDumpOnOutOfMemoryError"
JAVA_OPTS="$JAVA_OPTS -XX:HeapDumpPath=/opt/logs/cobar/java.hprof"
JAVA_OPTS="$JAVA_OPTS -XX:ErrorFile=/opt/logs/cobar/hs_err_pid%p.log"
```



http://blog.csdn.net/matt8/article/details/52298397

https://www.ibm.com/developerworks/cn/java/j-lo-jvm-optimize-experience/index.html#icomments

https://www.ibm.com/developerworks/cn/java/j-lo-JVMGarbageCollection/#icomments

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

http://www.importnew.com/15311.html
G1垃圾回收