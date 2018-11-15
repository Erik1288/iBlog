---
title: JVM-CMS
date: 2018-10-12 10:06:25
tags: JVM
---

##
https://blog.csdn.net/foolishandstupid/article/details/77430875

ParNew+CMS是Hotspot JVM中一对经典的GC组合，其中ParNew负责Young区的GC，CMS负责Old区的GC。

JVM启动会调用`Universe::initialize_heap`初始化Heap，如果JVM启动参数中指定了`-XX:+UseConcMarkSweepGC`，初始化堆时会创建CMS收集器（cms colloector）。
``` GenCollectedHeap::initialize()
if (collector_policy()->is_concurrent_mark_sweep_policy()) {
    bool success = create_cms_collector();
    if (!success) return JNI_ENOMEM;
}
```
忽略下中间部分细节，通过调试代码，最终会JVM会在`ConcurrentMarkSweepThread`的构造函数中启动该线程。JVM中C++对线程的抽象与Java非常类似，启动线程后，内核调度到该线程时，会执行它的run()方法。
``` 
// Create & start a CMS thread for this CMS collector
  _cmsThread = ConcurrentMarkSweepThread::start(this);


```
Stacktrace展示了CMS线程的启动时机。
```
ConcurrentGCThread::create_and_start concurrentGCThread.cpp:41
ConcurrentMarkSweepThread::ConcurrentMarkSweepThread concurrentMarkSweepThread.cpp:69
ConcurrentMarkSweepThread::ConcurrentMarkSweepThread concurrentMarkSweepThread.cpp:49
ConcurrentMarkSweepThread::start concurrentMarkSweepThread.cpp:107
CMSCollector::CMSCollector concurrentMarkSweepGeneration.cpp:592
CMSCollector::CMSCollector concurrentMarkSweepGeneration.cpp:490
GenCollectedHeap::create_cms_collector genCollectedHeap.cpp:829
GenCollectedHeap::initialize genCollectedHeap.cpp:144
Universe::initialize_heap universe.cpp:762
universe_init universe.cpp:672
init_globals init.cpp:110
Threads::create_vm thread.cpp:3630
JNI_CreateJavaVM_inner jni.cpp:3937
JNI_CreateJavaVM jni.cpp:4032
InitializeJVM
JavaMain
```


通过简单的分析，会发现`ConcurrentMarkSweepThread`自己没有run()，但是它的父类`ConcurrentGCThread`中有，如下所示，run_service是一个纯虚函数（类似Java的抽象方法），自然在运行时会指向子类`ConcurrentMarkSweepThread`对`run_service`的实现。
```
void ConcurrentGCThread::run() {
  initialize_in_thread();
  wait_for_universe_init();

  run_service(); // virtual void run_service() = 0; 虚函数，指向子类ConcurrentMarkSweepThread的实现

  terminate();
}
```

在`run_service`中`while (true)`的形式异常明显，暗示着这是一个生命周期很长的线程，只要没有被terminate，就会永远在后台执行。`ConcurrentMarkSweepThread`会在它的生命周期内，默认每隔`2000ms`就检查下，进程需不需要进行`background cms gc`。`sleepBeforeNextCycle()`描述了为什么是`2000ms`和什么样的情况下会需要GC，而`_collector->collect_in_background`会描述什么是`background`式的CMS。
```
void ConcurrentMarkSweepThread::run_service() {
  assert(this == cmst(), "just checking");

  if (BindCMSThreadToCPU && !os::bind_to_processor(CPUForCMSThread)) {
    log_warning(gc)("Couldn't bind CMS thread to processor " UINTX_FORMAT, CPUForCMSThread);
  }

  while (!should_terminate()) {
    sleepBeforeNextCycle();
    if (should_terminate()) break;
    GCIdMark gc_id_mark;
    GCCause::Cause cause = _collector->_full_gc_requested ?
      _collector->_full_gc_cause : GCCause::_cms_concurrent_mark;
    _collector->collect_in_background(cause);
  }

  // Check that the state of any protocol for synchronization
  // between background (CMS) and foreground collector is "clean"
  // (i.e. will not potentially block the foreground collector,
  // requiring action by us).
  verify_ok_to_terminate();
}
```
`CMSWaitDuration`默认宏定义为2000，调用`wait_on_cms_lock_for_scavenge(CMSWaitDuration)`后让当前线程等待，注意下它的注释，有3种情况当前的线程会被唤醒，
 下一次`synchronous GC`
 一个并发Full gc请求
 2000ms的超时到达
当线程醒后，只有当满足`_collector->shouldConcurrentCollect()`才能跳出当前调用，才能在外层真正调用CMS。
```
void ConcurrentMarkSweepThread::sleepBeforeNextCycle() {
  while (!should_terminate()) {
    if(CMSWaitDuration >= 0) {
      // Wait until the next synchronous GC, a concurrent full gc
      // request or a timeout, whichever is earlier.
      wait_on_cms_lock_for_scavenge(CMSWaitDuration);
    } else {
      // Wait until any cms_lock event or check interval not to call shouldConcurrentCollect permanently
      wait_on_cms_lock(CMSCheckInterval);
    }
    // Check if we should start a CMS collection cycle
    if (_collector->shouldConcurrentCollect()) {
      return;
    }
    // .. collection criterion not yet met, let's go back
    // and wait some more
  }
}
```
以下有 种Case会返回。重点分析下Case 
``` 
bool CMSCollector::shouldConcurrentCollect() {
  // Case 1
  if (_full_gc_requested) {
    log.print("CMSCollector: collect because of explicit  gc request (or GCLocker)");
    return true;
  }
  ...
  // If the estimated time to complete a cms collection (cms_duration())
  // is less than the estimated time remaining until the cms generation
  // is full, start a collection.
  if (!UseCMSInitiatingOccupancyOnly) {
    if (stats().valid()) {
      // Case 2
      if (stats().time_until_cms_start() == 0.0) {
        return true;
      }
    } else {
      // We want to conservatively collect somewhat early in order
      // to try and "bootstrap" our CMS/promotion statistics;
      // this branch will not fire after the first successful CMS
      // collection because the stats should then be valid.
      // Case 3
      if (_cmsGen->occupancy() >= _bootstrap_occupancy) {
        log.print(" CMSCollector: collect for bootstrapping statistics: occupancy = %f, boot occupancy = %f",
                  _cmsGen->occupancy(), _bootstrap_occupancy);
        return true;
      }
    }
  }

  // Otherwise, we start a collection cycle if
  // old gen want a collection cycle started. Each may use
  // an appropriate criterion for making this decision.
  // XXX We need to make sure that the gen expansion
  // criterion dovetails well with this. XXX NEED TO FIX THIS
  // Case 4
  if (_cmsGen->should_concurrent_collect()) {
    log.print("CMS old gen initiated");
    return true;
  }

  // We start a collection if we believe an incremental collection may fail;
  // this is not likely to be productive in practice because it's probably too
  // late anyway.
  GenCollectedHeap* gch = GenCollectedHeap::heap();
  assert(gch->collector_policy()->is_generation_policy(),
         "You may want to check the correctness of the following");
  // Case 5
  if (gch->incremental_collection_will_fail(true /* consult_young */)) {
    log.print("CMSCollector: collect because incremental collection will fail ");
    return true;
  }

  // Case 6
  if (MetaspaceGC::should_concurrent_collect()) {
    log.print("CMSCollector: collect for metadata allocation ");
    return true;
  }

  // CMSTriggerInterval starts a CMS cycle if enough time has passed.
  if (CMSTriggerInterval >= 0) {
    // Case 7
    if (CMSTriggerInterval == 0) {
      // Trigger always
      return true;
    }

    // Check the CMS time since begin (we do not check the stats validity
    // as we want to be able to trigger the first CMS cycle as well)
    // Case 8
    if (stats().cms_time_since_begin() >= (CMSTriggerInterval / ((double) MILLIUNITS))) {
      if (stats().valid()) {
        log.print("CMSCollector: collect because of trigger interval (time since last begin %3.7f secs)",
                  stats().cms_time_since_begin());
      } else {
        log.print("CMSCollector: collect because of trigger interval (first collection)");
      }
      return true;
    }
  }

  return false;
}
```


