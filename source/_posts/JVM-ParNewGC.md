---
title: JVM-ParNewGC
date: 2018-10-12 10:05:19
tags:
---

```
ParNewGeneration::collect(bool, bool, unsigned long, bool) parNewGeneration.cpp:884
GenCollectedHeap::collect_generation(Generation*, bool, unsigned long, bool, bool, bool, bool) genCollectedHeap.cpp:385
GenCollectedHeap::do_collection(bool, bool, unsigned long, bool, GenCollectedHeap::GenerationType) genCollectedHeap.cpp:471
GenCollectorPolicy::satisfy_failed_allocation(unsigned long, bool) collectorPolicy.cpp:738
GenCollectedHeap::satisfy_failed_allocation(unsigned long, bool) genCollectedHeap.cpp:554
VM_GenCollectForAllocation::doit() vmGCOperations.cpp:163
VM_Operation::evaluate() vm_operations.cpp:66
VMThread::evaluate_operation(VM_Operation*) vmThread.cpp:348
VMThread::loop() vmThread.cpp:470
VMThread::run() vmThread.cpp:262
thread_native_entry(Thread*) os_bsd.cpp:720
_pthread_body 0x00007fff9cc8193b
_pthread_start 0x00007fff9cc81887
thread_start 0x00007fff9cc8108d
```