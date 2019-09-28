---
title: JVM-TLAB
date: 2018-11-15 10:32:24
tags: JVM
---


### 疑问
按照我的理解，TLAB应该是JavaThread在申请内存时，在JavaThread Local上直接分配内存，如果TLAB的内存也不够了，那么再给该JavaThread分配TLAB。那么，Java中new Thread()难道还需要创建TLAB？这样这个操作不是显得有点重么？



在JVM的线程类定义中，有一个TLAB字段
``` thread.hpp
class Thread: public ThreadShadow {
    private:
    ThreadLocalAllocBuffer _tlab;                 // Thread-local eden
}
```


### TLAB从Eden区划出范围
```
ThreadLocalAllocBuffer::initialize threadLocalAllocBuffer.cpp:182
ThreadLocalAllocBuffer::fill threadLocalAllocBuffer.cpp:173
CollectedHeap::allocate_from_tlab_slow collectedHeap.cpp:322
CollectedHeap::allocate_from_tlab collectedHeap.inline.hpp:203
CollectedHeap::common_mem_allocate_noinit collectedHeap.inline.hpp:141
CollectedHeap::common_mem_allocate_init collectedHeap.inline.hpp:190
CollectedHeap::array_allocate collectedHeap.inline.hpp:241
TypeArrayKlass::allocate_common typeArrayKlass.cpp:109
TypeArrayKlass::allocate typeArrayKlass.hpp:67
oopFactory::new_intArray oopFactory.hpp:50
SystemDictionary::initialize systemDictionary.cpp:2078
Universe::genesis universe.cpp:317
universe2_init universe.cpp:977
init_globals init.cpp:122
Threads::create_vm thread.cpp:3630
JNI_CreateJavaVM_inner jni.cpp:3937
JNI_CreateJavaVM jni.cpp:4032
InitializeJVM 0x0000000100005e50
JavaMain 0x000000010000502c
_pthread_body 0x00007fff942c793b
_pthread_start 0x00007fff942c7887
thread_start 0x00007fff942c708d
<unknown> 0x0000000000000000
```