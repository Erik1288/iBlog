---
title: JVM-GC
date: 2018-08-11 15:20:17
tags:
---

### 写得很精彩
http://blog.chriscs.com/2017/06/20/g1-vs-cms/

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