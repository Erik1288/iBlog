---
title: RocketMQ-Performance-tunning
date: 2018-09-22 15:25:34
tags:
---


### 红帽linux性能调优 参考
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html-single/performance_tuning_guide/index


### mq4最佳实践，极佳
http://jm.taobao.org/2017/03/23/20170323/
http://jm.taobao.org/2017/01/26/20170126/
```
RocketMQ通过GC调优后最终采取的GC参数如下所示，供大家参考。

-server -Xms8g -Xmx8g -Xmn4g
-XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25
-XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0
-XX:SurvivorRatio=8 -XX:+DisableExplicitGC
-verbose:gc -Xloggc:/dev/shm/mq_gc_%p.log -XX:+PrintGCDetails
-XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime
-XX:+PrintAdaptiveSizePolicy
-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m
可以看出，我们最终全部切换到了G1，16年双十一线上MetaQ集群采用的也是这一组参数，基本上GC时间能控制在20ms以内（一些超大的共享集群除外）。

对于G1，官方推荐使用该-XX:MaxGCPauseMillis设置目标暂停时间，不要手动指定-Xmn和-XX:NewRatio，但我们在实测中发现，如果指定过小的目标停顿时间(10ms)，G1会将新生代调整为很小，导致YGC更加频繁，老年代用得更快，所有还是手动指定了-Xmn为4g，在GC频率不高的情况下完成了10ms的目标停顿时间，这里也说明有时候一些通用的调优经验并不适用于所有的产品场景，需要更多的测试才能找到最合适的调优方法，往往需要另辟蹊径。
```
但是事实是，我采用了这组参数，G1却最后扩大了New区，导致经常young gc停顿超过10ms，说明在极致的性能调优前，不能用官方推荐的参数，只能自己反复修改GC参数然后再测试。
```
JAVA_OPT="-server -XX:+UseG1GC -Xms8g -Xmx8g -Xss256k -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m"

JAVA_OPT="$JAVA_OPT -XX:MaxDirectMemorySize=30g"

JAVA_OPT="$JAVA_OPT -XX:+PrintSafepointStatistics"
JAVA_OPT="$JAVA_OPT -XX:PrintSafepointStatisticsCount=1"
JAVA_OPT="$JAVA_OPT -XX:+UnlockDiagnosticVMOptions"
JAVA_OPT="$JAVA_OPT -XX:-DisplayVMOutput"
JAVA_OPT="$JAVA_OPT -XX:+LogVMOutput"
JAVA_OPT="$JAVA_OPT -XX:LogFile=/opt/logs/rocketmq/vm.log"

JAVA_OPT="$JAVA_OPT -XX:MaxGCPauseMillis=10"
JAVA_OPT="$JAVA_OPT -XX:ParallelGCThreads=8"
JAVA_OPT="$JAVA_OPT -XX:ConcGCThreads=2"
JAVA_OPT="$JAVA_OPT -XX:SoftRefLRUPolicyMSPerMB=0"
JAVA_OPT="$JAVA_OPT -XX:+ParallelRefProcEnabled"
JAVA_OPT="$JAVA_OPT -XX:+DisableExplicitGC"
JAVA_OPT="$JAVA_OPT -XX:-UseLargePages"
JAVA_OPT="$JAVA_OPT -verbose:gc"
JAVA_OPT="$JAVA_OPT -Xloggc:/opt/logs/rocketmq/%p_gc.log"

JAVA_OPT="$JAVA_OPT -XX:+AggressiveOpts"
JAVA_OPT="$JAVA_OPT -XX:-UseBiasedLocking"
JAVA_OPT="$JAVA_OPT -XX:+UseFastAccessorMethods"
JAVA_OPT="$JAVA_OPT -XX:+PrintGCDetails"
JAVA_OPT="$JAVA_OPT -XX:+PrintGCDateStamps"
JAVA_OPT="$JAVA_OPT -XX:+PrintGCApplicationStoppedTime"
JAVA_OPT="$JAVA_OPT -XX:+PrintAdaptiveSizePolicy"
JAVA_OPT="$JAVA_OPT -XX:+PrintTenuringDistribution"
JAVA_OPT="$JAVA_OPT -XX:+PrintHeapAtGC"
JAVA_OPT="$JAVA_OPT -XX:+PrintFlagsFinal"
JAVA_OPT="$JAVA_OPT -XX:+PrintPromotionFailure"
JAVA_OPT="$JAVA_OPT -XX:+PrintReferenceGC"

JAVA_OPT="$JAVA_OPT -XX:+UseGCLogFileRotation"
JAVA_OPT="$JAVA_OPT -XX:NumberOfGCLogFiles=5"
JAVA_OPT="$JAVA_OPT -XX:GCLogFileSize=30m"
JAVA_OPT="$JAVA_OPT -XX:-OmitStackTraceInFastThrow"
JAVA_OPT="$JAVA_OPT -XX:+AlwaysPreTouch"
JAVA_OPT="$JAVA_OPT -XX:MaxTenuringThreshold=15"
JAVA_OPT="$JAVA_OPT -XX:+HeapDumpOnOutOfMemoryError"
JAVA_OPT="$JAVA_OPT -XX:HeapDumpPath=/opt/logs/rocketmq/java.hprof"
JAVA_OPT="$JAVA_OPT -XX:ErrorFile=/opt/logs/rocketmq/hs_err_pid%p.log"
```



### 是否用ReentrantLock
```
/**
 经过多人测试，在竞争非常激烈的情况下，用ReentrantLock性能会好过SpinLock，注意可以适当提高消息发送的线程数量
 * introduced since 4.0.x. Determine whether to use mutex reentrantLock when putting message.<br/>
 * By default it is set to false indicating using spin lock when putting message.
 */
private boolean useReentrantLockWhenPutMessage = false;
int sendMessageThreadPoolNums = 1; //16 + Runtime.getRuntime().availableProcessors() * 4;
```
如果将useReentrantLockWhenPutMessage设置为false（默认就是false），那RocketMQ会使用自旋锁。这个加锁的位置是Append到CommitLog，如果Append线程比较多，并且并发量大，自旋锁会大量在CPU上无效自旋，从而导致CPU Loader高居不下。所以高并发下，使用synchronized或者ReentrantLock是最佳的，但是由于RocketMQ没有提供synchronized这个选项，所以只能选择ReentrantLock。


### 
```
// Whether check the CRC32 of the records consumed.
// This ensures no on-the-wire or on-disk corruption to the messages occurred.
// This check adds some overhead,so it may be disabled in cases seeking extreme performance.
private boolean checkCRCOnRecover = true;
```

```
private boolean warmMapedFileEnable = false;
```
RocketMQ中每一个CommitLog都有固定1GB的大小，当某一个文件写完了，再去创建另一个文件，这个过程明显会成为性能瓶颈，所以RocketMQ的做法是每次在创建文件的时候，异步多创建几个文件来进行备用，这个想法也是非常直觉化的，但是预先创建了文件，往里边写数据还是会比较慢，由于mmap只是做了虚拟内存和文件的映射，但是虚拟内存和物理内存还没有关联上，在真实写的时候会产生多次缺页异常。如果将warmMapedFileEnable设置为true，RocketMQ会在创建完文件后，对文件进行预热（按照page为单位写0值，mlock，madvise），从而在真正写入时，能达到极致的刷盘效果。


###
```
    @ImportantField
    private boolean transientStorePoolEnable = false;
```
RocketMQ在消息落盘实践的最大创新点，这个机制在kafka上没有。TransientStorePool只有在异步刷盘时才能开启，是在RocketMQ启动时，会预留一个对外内存池，刷盘时，首先将ByteBuffer写入TransientStorePool，只要刷进内存，就给Producer一个成功的响应。使用了TransientStorePool，刷盘避免了使用mmap刷Page Cache，也就避免了需要Linux加锁回收内存，将刷盘的毛刺降到最低。

但是，开启TransientStorePool后有一个影响，那就是当RocketMQ进程Crash，已经通知了Producer刷盘成功，但是还没有最终调用fileChannel.force()的消息将会丢失，本来就是设置异步刷盘，所以这个就是在性能和数据可靠性间做权衡。下图就是TransientStorePool开启和未开启的原理图。

###
```
    private boolean transferMsgByHeap = true;
```
传统消息消费时，数据从Page Cache中拿出来后，被Copy到JVM堆内存，然后再传给JVM堆外的Socket通道，整个过程是数据在OS->JVM堆内->OS流转。将transferMsgByHeap设置为false后，就意味着数据从Page Cache中拿出来后，不会被Copy一份到JVM堆内存，直接从Page Cache到Socket通道，这样就使用到了Zero-Copy功能。下图是RocketMQ源码位置，感兴趣的同学可以读一读这段逻辑。


###
```
    private boolean slaveReadEnable = false;
```
当消费者需要读取的数据与现在的Offset偏差比较大（比较久远），不管要读取多少数据，现代操作系统都会使用「Page Cache」，这样就会强制回收一部分在文件末端的Cache，内存分配的速度如果没有赶上内存回收的速度，在刷盘时就会产生毛刺，并且如果改消费者稳定从久远数据开始读取，会使当前的Broker Master在消息落盘和消息消费都产生影响。将slaveReadEnable设置为true后，如果读取的数据比较久远，Broker会推荐Consumer去Slave读取，从而缓解Master的压力。

###
```
    private boolean serverPooledByteBufAllocatorEnable = true;
```

###
```
    /**
     * make make install
     *
     *
     * ../glibc-2.10.1/configure \ --prefix=/usr \ --with-headers=/usr/include \
     * --host=x86_64-linux-gnu \ --build=x86_64-pc-linux-gnu \ --without-gd
     */
    private boolean useEpollNativeSelector = false;
```

这块代码是否可以优化？
```
private byte[] readGetMessageResult(final GetMessageResult getMessageResult, final String group, final String topic,
    final int queueId) {

    // 这里频繁的申请堆内内存，可以进行优化
    final ByteBuffer byteBuffer = ByteBuffer.allocate(getMessageResult.getBufferTotalSize());

    long storeTimestamp = 0;
    try {
        List<ByteBuffer> messageBufferList = getMessageResult.getMessageBufferList();
        for (ByteBuffer bb : messageBufferList) {

            byteBuffer.put(bb);
            storeTimestamp = bb.getLong(MessageDecoder.MESSAGE_STORE_TIMESTAMP_POSTION);
        }
    } finally {
        getMessageResult.release();
    }

    this.brokerController.getBrokerStatsManager().recordDiskFallBehindTime(group, topic, queueId, this.brokerController.getMessageStore().now() - storeTimestamp);
    return byteBuffer.array();
}
```