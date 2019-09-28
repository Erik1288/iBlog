---
title: JVM-CMS
date: 2018-10-12 10:06:25
tags: JVM
---


CollectedHeap
  GenCollectedHeap
  G1CollectedHeap
  ParallelScavengeHeap


### 还是杜兄的文章
https://www.jianshu.com/p/78017c8b8e0f


### CMS不是Full GC吧？

### R大
作者：RednaxelaFX
链接：https://www.zhihu.com/question/63785052/answer/216407946
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

不知道题主是读了什么资料留下了“Java 8或者G1开始HotSpot才引入了card table / remembered set”的印象。但这个印象是错误的。HotSpot VM的GC，直到G1之前为止，其最基础的设计全部都是源自UC Berkeley在1980年代中期做的Berkeley Smalltalk。相关论文是David Ungar的Generation Scavenging。这个设计沿着 Berkeley Smalltalk -> Self -> Strongtalk -> HotSpot这样的血缘一路传承到了HotSpot VM里。这是一个分代式GC的设计，其中young generation分为1个eden区和2个survivor区，采用copying GC（又名scavenging）算法来收集。跨分代（old -> young）的引用通过粗粒度的card table来记录。回来看HotSpot VM里的GC实现，除了G1之外全部都是完全按照Berkeley Smalltalk的这个原始设计来做的。改变的只是经过多年的发展，把young GC从serial演进到了parallel，而在old generation上除了serial mark-compact之外也额外增加了parallel mark-compact（Parallel Old）和mostly-concurrent mark-sweep（CMS）。但是heap layout以及跨分代的引用的记录方式是完全没有改变的。反而G1是完全打破了这种heap layout设计，在整个GC堆上都采用了新的region-based设计——把GC堆划分为很多（例如1024）个相同大小的region，然后在上面动态地、逻辑地选择其中一些region作为eden region，一些作为survivor region，一些作为old generation region。其中属于young generation的region不需要记录outgoing reference的信息（或者从其它young generation出发的incoming reference的信息），因为这些region会在每次GC的时候都被完全扫描——G1里的三种GC模式，young、mixed和full，都保证会完全扫描young generation里的所有region。而属于old generation的region则可以被单独选出来放到mixed GC里去做收集，这些region就需要记录outgoing和incoming reference的信息。其中outgoing reference是直接通过跟HotSpot里其它GC一样的card table来记录的，而incoming reference则是通过G1特有的per-region remembered set结合card table来记录的。为此G1比以前的HotSpot GC新增了一种remembered set的设计，但是同时也使用了跟以前HotSpot VM里其它GC完全一样的card table。


### 里面关于CMS产生碎片的图片很生动
https://blog.csdn.net/zhanggang807/article/details/45956325


``` 
CMSCollector::collect_in_background(GCCause::Cause cause) {
  ...
  while (_collectorState != Idling) {
    log_debug(gc, state)("Thread " INTPTR_FORMAT " in CMS state %d",
                         p2i(Thread::current()), _collectorState);
    // The foreground collector
    //   holds the Heap_lock throughout its collection.
    //   holds the CMS token (but not the lock)
    //     except while it is waiting for the background collector to yield.
    //
    // The foreground collector should be blocked (not for long)
    //   if the background collector is about to start a phase
    //   executed with world stopped.  If the background
    //   collector has already started such a phase, the
    //   foreground collector is blocked waiting for the
    //   Heap_lock.  The stop-world phases (InitialMarking and FinalMarking)
    //   are executed in the VM thread.
    //
    // The locking order is
    //   PendingListLock (PLL)  -- if applicable (FinalMarking)
    //   Heap_lock  (both this & PLL locked in VM_CMS_Operation::prologue())
    //   CMS token  (claimed in
    //                stop_world_and_do() -->
    //                  safepoint_synchronize() -->
    //                    CMSThread::synchronize())

    {
      // Check if the FG collector wants us to yield.
      CMSTokenSync x(true); // is cms thread
      if (waitForForegroundGC()) {
        // We yielded to a foreground GC, nothing more to be
        // done this round.
        assert(_foregroundGCShouldWait == false, "We set it to false in "
               "waitForForegroundGC()");
        log_debug(gc, state)("CMS Thread " INTPTR_FORMAT " exiting collection CMS state %d",
                             p2i(Thread::current()), _collectorState);
        return;
      } else {
        // The background collector can run but check to see if the
        // foreground collector has done a collection while the
        // background collector was waiting to get the CGC_lock
        // above.  If yes, break so that _foregroundGCShouldWait
        // is cleared before returning.
        if (_collectorState == Idling) {
          break;
        }
      }
    }

    assert(_foregroundGCShouldWait, "Foreground collector, if active, "
      "should be waiting");

    switch (_collectorState) {
      case InitialMarking:
        {
          ReleaseForegroundGC x(this);
          stats().record_cms_begin();
          VM_CMS_Initial_Mark initial_mark_op(this);
          VMThread::execute(&initial_mark_op);
        }
        // The collector state may be any legal state at this point
        // since the background collector may have yielded to the
        // foreground collector.
        break;
      case Marking:
        // initial marking in checkpointRootsInitialWork has been completed
        if (markFromRoots()) { // we were successful
          assert(_collectorState == Precleaning, "Collector state should "
            "have changed");
        } else {
          assert(_foregroundGCIsActive, "Internal state inconsistency");
        }
        break;
      case Precleaning:
        // marking from roots in markFromRoots has been completed
        preclean();
        assert(_collectorState == AbortablePreclean ||
               _collectorState == FinalMarking,
               "Collector state should have changed");
        break;
      case AbortablePreclean:
        abortable_preclean();
        assert(_collectorState == FinalMarking, "Collector state should "
          "have changed");
        break;
      case FinalMarking:
        {
          ReleaseForegroundGC x(this);

          VM_CMS_Final_Remark final_remark_op(this);
          VMThread::execute(&final_remark_op);
        }
        assert(_foregroundGCShouldWait, "block post-condition");
        break;
      case Sweeping:
        // final marking in checkpointRootsFinal has been completed
        sweep();
        assert(_collectorState == Resizing, "Collector state change "
          "to Resizing must be done under the free_list_lock");

      case Resizing: {
        // Sweeping has been completed...
        // At this point the background collection has completed.
        // Don't move the call to compute_new_size() down
        // into code that might be executed if the background
        // collection was preempted.
        {
          ReleaseForegroundGC x(this);   // unblock FG collection
          MutexLockerEx       y(Heap_lock, Mutex::_no_safepoint_check_flag);
          CMSTokenSync        z(true);   // not strictly needed.
          if (_collectorState == Resizing) {
            compute_new_size();
            save_heap_summary();
            _collectorState = Resetting;
          } else {
            assert(_collectorState == Idling, "The state should only change"
                   " because the foreground collector has finished the collection");
          }
        }
        break;
      }
      case Resetting:
        // CMS heap resizing has been completed
        reset_concurrent();
        assert(_collectorState == Idling, "Collector state should "
          "have changed");

        MetaspaceGC::set_should_concurrent_collect(false);

        stats().record_cms_end();
        // Don't move the concurrent_phases_end() and compute_new_size()
        // calls to here because a preempted background collection
        // has it's state set to "Resetting".
        break;
      case Idling:
      default:
        ShouldNotReachHere();
        break;
    }
    log_debug(gc, state)("  Thread " INTPTR_FORMAT " done - next CMS state %d",
                         p2i(Thread::current()), _collectorState);
    assert(_foregroundGCShouldWait, "block post-condition");
  }
  ...
}
```


