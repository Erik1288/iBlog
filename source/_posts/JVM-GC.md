---
title: JVM-GC
date: 2018-08-11 15:20:17
tags: JVM
---

### 写得很精彩
http://blog.chriscs.com/2017/06/20/g1-vs-cms/


### GC用到的算法
https://www.iecc.com/gclist/GC-algorithms.html

### GC Roots
1. 线程方法栈内引用 Student s = new Student() 中的 s
2. 静态引用 static
3. 常量引用 const
4. jni线程方法栈内引用

### minor gc & major gc & full gc， what are the definitions of them?
https://plumbr.io/blog/garbage-collection/minor-gc-vs-major-gc-vs-full-gc

### oracle's blog -- CMS GC log
https://blogs.oracle.com/poonam/understanding-cms-gc-logs

### 分代内存比例
https://blog.csdn.net/Muyundefeng/article/details/72667863

### cms与g1的区别
http://www.woowen.com/java/2016/12/10/G1%20CMS%E5%8C%BA%E5%88%AB/

https://www.jianshu.com/p/35cd012eeb8c

### g1gc log format
https://blogs.oracle.com/poonam/understanding-g1-gc-logs

### 画图画起来，这个GC原理画得很不错，可以借鉴
https://www.jianshu.com/p/314272e6d35b

### 调试GC源码
```
// The same dtrace probe can't be inserted in two different files, so we
// have to call it here, so it's only in one file.  Can't create new probes
// for the other file anymore.   The dtrace probes have to remain stable.
void VM_GC_Operation::notify_gc_begin(bool full) {
  HOTSPOT_GC_BEGIN(
                   full);
  HS_DTRACE_WORKAROUND_TAIL_CALL_BUG();
}

VMThread::execute(VM_Operation*) vmThread.cpp:594
G1CollectedHeap::do_collection_pause(unsigned long, unsigned int, bool*, GCCause::Cause) g1CollectedHeap.cpp:2741
G1CollectedHeap::attempt_allocation_slow(unsigned long, unsigned char, unsigned int*, unsigned int*) g1CollectedHeap.cpp:588
G1CollectedHeap::attempt_allocation(unsigned long, unsigned int*, unsigned int*) g1CollectedHeap.cpp:853
G1CollectedHeap::mem_allocate(unsigned long, bool*) g1CollectedHeap.cpp:485
CollectedHeap::common_mem_allocate_noinit(KlassHandle, unsigned long, Thread*) collectedHeap.inline.hpp:149
CollectedHeap::common_mem_allocate_init(KlassHandle, unsigned long, Thread*) collectedHeap.inline.hpp:190
CollectedHeap::array_allocate(KlassHandle, int, int, Thread*) collectedHeap.inline.hpp:241
TypeArrayKlass::allocate_common(int, bool, Thread*) typeArrayKlass.cpp:109
TypeArrayKlass::allocate(int, Thread*) typeArrayKlass.hpp:67
oopFactory::new_typeArray(BasicType, int, Thread*) oopFactory.cpp:56
Runtime1::new_type_array(JavaThread*, Klass*, int) c1_Runtime1.cpp:351
<unknown> 0x000000010d498ec9
<unknown> 0x000000010d84f0ae
<unknown> 0x000000010d2244a3
<unknown> 0x000000010d2199f1
JavaCalls::call_helper(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) javaCalls.cpp:410
os::os_exception_wrapper(void (*)(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*), JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) os_bsd.cpp:3682
JavaCalls::call(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) javaCalls.cpp:306
jni_invoke_static(JNIEnv_*, JavaValue*, _jobject*, JNICallType, _jmethodID*, JNI_ArgumentPusher*, Thread*) jni.cpp:1119
::jni_CallStaticVoidMethod(JNIEnv *, jclass, jmethodID, ...) jni.cpp:1989
JavaMain 0x0000000105b69c90
_pthread_body 0x00007fffc9ad493b
_pthread_start 0x00007fffc9ad4887
thread_start 0x00007fffc9ad408d
```


### 从两个角度来理解GC
1. GC线程本身是串行的单线程，还是并行的多线程
2. GC线程和应用线程是并行的，还是应用线程会完全停止


疑问
eden区域的垃圾对象和存活对象是怎么分辨出来的？

发生了一次YGC，为什么O的占用率变小了？
```

[jump@mq4-broker001 ~]$ sudo jstat -gcutil 20003 1000 100
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT 
  0.00 100.00  92.16   3.57  97.01  89.12   5339   37.997     0    0.000   37.997
  0.00 100.00  92.54   3.57  97.01  89.12   5339   37.997     0    0.000   37.997
  0.00 100.00  92.91   3.57  97.01  89.12   5339   37.997     0    0.000   37.997
  0.00 100.00  93.66   3.57  97.01  89.12   5339   37.997     0    0.000   37.997
  0.00 100.00  94.03   3.57  97.01  89.12   5339   37.997     0    0.000   37.997
  0.00 100.00  94.78   3.57  97.01  89.12   5339   37.997     0    0.000   37.997
  0.00 100.00   1.12   3.54  97.01  89.12   5340   38.005     0    0.000   38.005
  0.00 100.00   1.49   3.54  97.01  89.12   5340   38.005     0    0.000   38.005
  0.00 100.00   1.87   3.54  97.01  89.12   5340   38.005     0    0.000   38.005
  0.00 100.00   2.61   3.54  97.01  89.12   5340   38.005     0    0.000   38.005
  0.00 100.00   3.36   3.54  97.01  89.12   5340   38.005     0    0.000   38.005
  0.00 100.00   3.36   3.54  97.01  89.12   5340   38.005     0    0.000   38.005
  0.00 100.00   3.36   3.54  97.01  89.12   5340   38.005     0    0.000   38.005
  0.00 100.00   4.48   3.54  97.01  89.12   5340   38.005     0    0.000   38.005
  
  
  [jump@mq4-broker001 bin]$ sudo jinfo -flags 20003
  Attaching to process ID 20003, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 25.181-b13
  Non-default VM flags: -XX:+AlwaysPreTouch -XX:CICompilerCount=4 -XX:ConcGCThreads=8 -XX:+DisableExplicitGC -XX:G1HeapRegionSize=16777216 -XX:G1ReservePercent=25 -XX:GCLogFileSize=31457280 -XX:InitialHeapSize=8589934592 -XX:InitiatingHeapOccupancyPercent=30 -XX:MarkStackSize=4194304 -XX:MaxDirectMemorySize=32212254720 -XX:MaxHeapSize=8589934592 -XX:MaxNewSize=4294967296 -XX:MinHeapDeltaBytes=16777216 -XX:NewSize=4294967296 -XX:NumberOfGCLogFiles=5 -XX:-OmitStackTraceInFastThrow -XX:ParallelGCThreads=8 -XX:+ParallelRefProcEnabled -XX:+PrintAdaptiveSizePolicy -XX:+PrintFlagsFinal -XX:+PrintGC -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC -XX:+PrintReferenceGC -XX:+PrintTenuringDistribution -XX:SoftRefLRUPolicyMSPerMB=0 -XX:SurvivorRatio=8 -XX:-UseBiasedLocking -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC -XX:+UseGCLogFileRotation -XX:-UseLargePages 
  Command line:  -Xms8g -Xmx8g -Xmn4g -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0 -XX:SurvivorRatio=8 -XX:+DisableExplicitGC -XX:+ParallelRefProcEnabled -XX:ConcGCThreads=8 -XX:ParallelGCThreads=8 -XX:+PrintFlagsFinal -XX:+PrintReferenceGC -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC -verbose:gc -Xloggc:/opt/logs/rocketmq/mq_gc_%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintAdaptiveSizePolicy -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m -XX:-OmitStackTraceInFastThrow -XX:+AlwaysPreTouch -XX:MaxDirectMemorySize=30g -XX:-UseLargePages -XX:-UseBiasedLocking -Djava.ext.dirs=/jre/lib/ext:/opt/app/rocketmq/bin/../lib
```