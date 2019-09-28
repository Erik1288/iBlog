---
title: JVM-Safepoint
date: 2018-10-04 16:22:34
tags: JVM
---

https://blog.csdn.net/column/details/talk-about-jvm.html

### 言简意赅
https://juejin.im/post/5c0cd964f265da616e4c4340

### 这篇写得很好
https://blog.csdn.net/iter_zc/article/details/41847887

### 这个要掌握
https://www.lmax.com/blog/staff-blogs/2015/08/05/jvm-guaranteed-safepoints/
http://www.importnew.com/16068.html
如果线程非常多，可以用`GuaranteedSafepointInterval`提高”保证安全点”的执行频率，这样会有效减少暂停。

### 江南白衣本衣
http://www.10tiao.com/html/698/201808/2247483961/1.html
http://calvin1978.blogcn.com/articles/safepoint.html

### stackoverflow上的safepoint问题
https://stackoverflow.com/questions/20134769/how-to-get-java-stacks-when-jvm-cant-reach-a-safepoint

### 知乎
https://www.zhihu.com/search?type=content&q=safepoint

怎么看这段日志，结合GC日志和Safepoint就明白了，注意下，最后一条GC是不会显示`Total time for which application threads`的
```
Heap after GC invocations=30 (full 0):
 garbage-first heap   total 18374656K, used 1386562K [0x000000035e800000, 0x000000035f004618, 0x00000007c0000000)
  region size 8192K, 14 young (114688K), 14 survivors (114688K)
 Metaspace       used 21800K, capacity 22150K, committed 22400K, reserved 1069056K
  class space    used 2333K, capacity 2416K, committed 2432K, reserved 1048576K
}
 [Times: user=0.28 sys=0.02, real=0.08 secs]
2018-12-18T16:10:18.180+0800: 3439.538: Total time for which application threads were stopped: 0.1329005 seconds, Stopping threads took: 0.0095877 seconds
2018-12-18T16:10:19.227+0800: 3440.586: Total time for which application threads were stopped: 0.0472667 seconds, Stopping threads took: 0.0095095 seconds
2018-12-18T16:10:20.273+0800: 3441.632: Total time for which application threads were stopped: 0.0460854 seconds, Stopping threads took: 0.0094629 seconds
2018-12-18T16:11:12.324+0800: 3493.682: Total time for which application threads were stopped: 0.0464605 seconds, Stopping threads took: 0.0094669 seconds
2018-12-18T16:11:18.371+0800: 3499.729: Total time for which application threads were stopped: 0.0466823 seconds, Stopping threads took: 0.0094960 seconds
2018-12-18T16:11:21.417+0800: 3502.775: Total time for which application threads were stopped: 0.0456646 seconds, Stopping threads took: 0.0093249 seconds
2018-12-18T16:11:26.463+0800: 3507.822: Total time for which application threads were stopped: 0.0463651 seconds, Stopping threads took: 0.0094908 seconds
2018-12-18T16:11:44.512+0800: 3525.870: Total time for which application threads were stopped: 0.0467351 seconds, Stopping threads took: 0.0094312 seconds
2018-12-18T19:09:18.204+0800: 14179.563: [GC pause (G1 Evacuation Pause) (young) 14179.572: [G1Ergonomics (CSet Construction) start choosing CSet, _pending_cards: 26650, predicted base time: 63.14 ms, remaining time: 0.00 ms, target pause time: 50.00 ms]
 14179.572: [G1Ergonomics (CSet Construction) add young regions to CSet, eden: 100 regions, survivors: 12 regions, predicted young region time: 8.18 ms]
 14179.572: [G1Ergonomics (CSet Construction) finish choosing CSet, eden: 100 regions, survivors: 12 regions, old: 0 regions, predicted pause time: 71.32 ms, target pause time: 50.00 ms]
, 0.0791549 secs]
   [Parallel Time: 39.9 ms, GC Workers: 8]
      [GC Worker Start (ms): Min: 14179586.9, Avg: 14179587.0, Max: 14179587.2, Diff: 0.2]
      [Ext Root Scanning (ms): Min: 18.2, Avg: 18.5, Max: 19.5, Diff: 1.2, Sum: 148.3]
      [Update RS (ms): Min: 12.6, Avg: 13.5, Max: 13.6, Diff: 1.0, Sum: 107.6]
         [Processed Buffers: Min: 2394, Avg: 2568.1, Max: 2899, Diff: 505, Sum: 20545]
      [Scan RS (ms): Min: 1.1, Avg: 1.1, Max: 1.2, Diff: 0.1, Sum: 9.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 6.2, Avg: 6.3, Max: 6.4, Diff: 0.2, Sum: 50.4]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 8]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.1, Max: 0.2, Diff: 0.1, Sum: 0.8]
      [GC Worker Total (ms): Min: 39.4, Avg: 39.5, Max: 39.7, Diff: 0.3, Sum: 316.3]
      [GC Worker End (ms): Min: 14179626.5, Avg: 14179626.6, Max: 14179626.6, Diff: 0.1]
   [Code Root Fixup: 0.1 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.4 ms]
   [Other: 38.8 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 3.6 ms]
      [Ref Enq: 0.2 ms]
      [Redirty Cards: 0.2 ms]
      [Humongous Register: 0.1 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.2 ms]
   [Eden: 800.0M(800.0M)->0.0B(784.0M) Survivors: 96.0M->112.0M Heap: 4178.0M(17.5G)->3408.0M(17.5G)]
Heap after GC invocations=137 (full 0):
 garbage-first heap   total 18374656K, used 3489792K [0x000000035e800000, 0x000000035f004618, 0x00000007c0000000)
  region size 8192K, 14 young (114688K), 14 survivors (114688K)
 Metaspace       used 21988K, capacity 22278K, committed 22400K, reserved 1069056K
  class space    used 2342K, capacity 2416K, committed 2432K, reserved 1048576K
}
 [Times: user=0.29 sys=0.00, real=0.09 secs]
```
```
3439.405: G1IncCollectionPause             [   24775          0              1    ]      [     0     0     9    33    83    ]  0
3440.538: no vm operation                  [   24775          0              0    ]      [     0     0     9    33     0    ]  0
3441.586: no vm operation                  [   24775          0              0    ]      [     0     0     9    33     0    ]  0
3493.636: no vm operation                  [   24775          0              0    ]      [     0     0     9    33     0    ]  0
3499.683: no vm operation                  [   24775          0              0    ]      [     0     0     9    33     0    ]  0
3502.729: no vm operation                  [   24775          0              0    ]      [     0     0     9    33     0    ]  0
3507.776: no vm operation                  [   24775          0              0    ]      [     0     0     9    33     0    ]  0
3525.823: no vm operation                  [   24775          0              0    ]      [     0     0     9    33     0    ]  0
14179.520: G1IncCollectionPause             [   24775          0              0    ]      [     0     0     6    33    85    ]  0
```


### 奇怪日志 Stopping threads took: 1.7947183 seconds
```
{Heap before GC invocations=64561 (full 0):
 garbage-first heap   total 8388608K, used 5195988K [0x00000005c0000000, 0x00000005c1001000, 0x00000007c0000000)
  region size 16384K, 256 young (4194304K), 1 survivors (16384K)
 Metaspace       used 24248K, capacity 25174K, committed 25472K, reserved 1073152K
  class space    used 2341K, capacity 2570K, committed 2688K, reserved 1048576K
2018-12-21T00:43:08.483+0800: 3836207.662: [GC pause (G1 Evacuation Pause) (young)
Desired survivor size 268435456 bytes, new threshold 15 (max 15)
- age   1:    1204016 bytes,    1204016 total
- age   2:     116808 bytes,    1320824 total
- age   3:      22992 bytes,    1343816 total
- age   4:      31216 bytes,    1375032 total
- age   5:      18072 bytes,    1393104 total
- age   6:      21472 bytes,    1414576 total
- age   7:      14512 bytes,    1429088 total
- age   8:      16472 bytes,    1445560 total
- age   9:      13664 bytes,    1459224 total
- age  10:      12784 bytes,    1472008 total
- age  11:      12888 bytes,    1484896 total
- age  12:      11040 bytes,    1495936 total
- age  13:      35480 bytes,    1531416 total
- age  14:      11008 bytes,    1542424 total
- age  15:      11360 bytes,    1553784 total
 3836207.662: [G1Ergonomics (CSet Construction) start choosing CSet, _pending_cards: 3421, predicted base time: 6.10 ms, remaining time: 193
.90 ms, target pause time: 200.00 ms]
 3836207.662: [G1Ergonomics (CSet Construction) add young regions to CSet, eden: 255 regions, survivors: 1 regions, predicted young region t
ime: 0.99 ms]
 3836207.662: [G1Ergonomics (CSet Construction) finish choosing CSet, eden: 255 regions, survivors: 1 regions, old: 0 regions, predicted pau
se time: 7.09 ms, target pause time: 200.00 ms]
2018-12-21T00:43:08.487+0800: 3836207.666: [SoftReference, 0 refs, 0.0008333 secs]2018-12-21T00:43:08.487+0800: 3836207.666: [WeakReference, 0 refs, 0.0004842 secs]2018-12-21T00:43:08.488+0800: 3836207.667: [FinalReference, 36 refs, 0.0003903 secs]2018-12-21T00:43:08.488+0800: 3836207.667: [PhantomReference, 0 refs, 1 refs, 0.0011838 secs]2018-12-21T00:43:08.490+0800: 3836207.669: [JNI Weak Reference, 0.0000125 secs], 0.0076618 secs]
   [Parallel Time: 2.9 ms, GC Workers: 8]
      [GC Worker Start (ms): Min: 3836207662.5, Avg: 3836207662.6, Max: 3836207662.7, Diff: 0.2]
      [Ext Root Scanning (ms): Min: 0.5, Avg: 0.6, Max: 0.7, Diff: 0.2, Sum: 5.1]
      [Update RS (ms): Min: 0.7, Avg: 0.8, Max: 0.9, Diff: 0.2, Sum: 6.5]
         [Processed Buffers: Min: 3, Avg: 14.1, Max: 28, Diff: 25, Sum: 113]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.1, Sum: 0.4]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.9, Avg: 1.0, Max: 1.1, Diff: 0.1, Sum: 8.0]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 8]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.1, Sum: 0.3]
      [GC Worker Total (ms): Min: 2.4, Avg: 2.6, Max: 2.6, Diff: 0.2, Sum: 20.4]
      [GC Worker End (ms): Min: 3836207665.1, Avg: 3836207665.2, Max: 3836207665.2, Diff: 0.1]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.4 ms]
   [Other: 4.3 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 3.2 ms]
      [Ref Enq: 0.2 ms]
      [Redirty Cards: 0.2 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.3 ms]
   [Eden: 4080.0M(4080.0M)->0.0B(4080.0M) Survivors: 16.0M->16.0M Heap: 5074.2M(8192.0M)->994.1M(8192.0M)]
Heap after GC invocations=64562 (full 0):
 garbage-first heap   total 8388608K, used 1017975K [0x00000005c0000000, 0x00000005c1001000, 0x00000007c0000000)
  region size 16384K, 1 young (16384K), 1 survivors (16384K)
 Metaspace       used 24248K, capacity 25174K, committed 25472K, reserved 1073152K
  class space    used 2341K, capacity 2570K, committed 2688K, reserved 1048576K
}
 [Times: user=0.00 sys=0.00, real=0.01 secs]
2018-12-21T00:43:08.491+0800: 3836207.670: Total time for which application threads were stopped: 1.8036386 seconds, Stopping threads took: 1.7947183 seconds
```

### 疑问 page_trap_count=1 代表什么，系统有STW
         vmop                    [threads: total initially_running wait_to_block]    [time: spin block sync cleanup vmop] page_trap_count
380.671: G1IncCollectionPause             [     167          1              2    ]      [     0     0     0     0    14    ]  1

### 日志
-XX:+PrintSafepointStatistics
```
         vmop                    [threads: total initially_running wait_to_block]    [time: spin block sync cleanup vmop] page_trap_count
0.265: Deoptimize                       [       9          0              0    ]      [     0     0     0     0     0    ]  0
0.312: G1IncCollectionPause             [       9          0              0    ]      [     0     0     0     0    11    ]  0
0.366: Deoptimize                       [      10          0              0    ]      [     0     0     0     0     0    ]  0
0.369: Deoptimize                       [      10          0              0    ]      [     0     0     0     0     0    ]  0
0.376: G1IncCollectionPause             [      10          0              0    ]      [     0     0     0     0     6    ]  0
0.432: Deoptimize                       [      12          0              0    ]      [     0     0     0     0     0    ]  0
0.532: G1IncCollectionPause             [      12          0              0    ]      [     0     0     0     0     4    ]  0
0.644: G1IncCollectionPause             [      12          1              0    ]      [     0     0     0     0     8    ]  0
0.678: G1IncCollectionPause             [      12          0              0    ]      [     0     0     0     0     9    ]  0
0.715: G1IncCollectionPause             [      12          1              0    ]      [     0     0     0     0     8    ]  0
0.917: EnableBiasedLocking              [      12          0              0    ]      [     0     0     0     0     0    ]  0
0.961: G1IncCollectionPause             [      13          0              0    ]      [     0     0     0     0     4    ]  0
0.967: RevokeBias                       [      14          1              1    ]      [     2     0     2     0     0    ]  0
0.974: RevokeBias                       [      14          0              2    ]      [     0     0     0     0     0    ]  0
0.982: G1IncCollectionPause             [      14          1              2    ]      [     3     0     4     0     7    ]  0
1.003: G1IncCollectionPause             [      14          0              0    ]      [     0     0     0     0     3    ]  0
1.007: G1IncCollectionPause             [      14          0              0    ]      [     0     0     0     0     2    ]  0
1.010: G1IncCollectionPause             [      14          0              0    ]      [     0     0     0     0     5    ]  0
1.016: G1IncCollectionPause             [      14          0              1    ]      [     0     0     0     0     5    ]  0
1.028: G1IncCollectionPause             [      16          0              1    ]      [     0     0     0     0     8    ]  0
1.040: G1IncCollectionPause             [      16          0              2    ]      [     0     0     0     0    44    ]  0
1.086: CGC_Operation                    [      16          0              0    ]      [     0     0     0     0    13    ]  0
1.100: CGC_Operation                    [      16          0              0    ]      [     0     0     0     0     0    ]  0
1.110: G1IncCollectionPause             [      18          1              1    ]      [     0     0     0     0    67    ]  0
1.179: G1IncCollectionPause             [      20          0              0    ]      [     0     0     0     0    47    ]  0
1.228: G1IncCollectionPause             [      20          0              0    ]      [     0     0     0     0    39    ]  0
1.268: RevokeBias                       [      22          0              1    ]      [     0     0     0     0     0    ]  0
1.269: RevokeBias                       [      23          0              1    ]      [     0     0     0     0     0    ]  0
2.269: no vm operation                  [      25          0              0    ]      [     0     0     0     0     0    ]  0
6.665: RevokeBias                       [      28          0              0    ]      [     0     0     0     0     0    ]  0
```

### 源码
```
void VMThread::execute(VM_Operation* op) {
  Thread* t = Thread::current();

  if (!t->is_VM_thread()) {
    SkipGCALot sgcalot(t);    // avoid re-entrant attempts to gc-a-lot
    // JavaThread or WatcherThread
    bool concurrent = op->evaluate_concurrently();
    // only blocking VM operations need to verify the caller's safepoint state:
    if (!concurrent) {
      t->check_for_valid_safepoint_state(true);
    }

    // New request from Java thread, evaluate prologue
    if (!op->doit_prologue()) {
      return;   // op was cancelled
    }

    // Setup VM_operations for execution
    op->set_calling_thread(t, Thread::get_priority(t));

    // It does not make sense to execute the epilogue, if the VM operation object is getting
    // deallocated by the VM thread.
    bool execute_epilog = !op->is_cheap_allocated();
    assert(!concurrent || op->is_cheap_allocated(), "concurrent => cheap_allocated");

    // Get ticket number for non-concurrent VM operations
    int ticket = 0;
    if (!concurrent) {
      ticket = t->vm_operation_ticket();
    }

    // Add VM operation to list of waiting threads. We are guaranteed not to block while holding the
    // VMOperationQueue_lock, so we can block without a safepoint check. This allows vm operation requests
    // to be queued up during a safepoint synchronization.
    {
      VMOperationQueue_lock->lock_without_safepoint_check();
      bool ok = _vm_queue->add(op);
    op->set_timestamp(os::javaTimeMillis());
      VMOperationQueue_lock->notify();
      VMOperationQueue_lock->unlock();
      // VM_Operation got skipped
      if (!ok) {
        assert(concurrent, "can only skip concurrent tasks");
        if (op->is_cheap_allocated()) delete op;
        return;
      }
    }

    if (!concurrent) {
      // Wait for completion of request (non-concurrent)
      // Note: only a JavaThread triggers the safepoint check when locking
      MutexLocker mu(VMOperationRequest_lock);
      while(t->vm_operation_completed_count() < ticket) {
        VMOperationRequest_lock->wait(!t->is_Java_thread());
      }
    }

    if (execute_epilog) {
      op->doit_epilogue();
    }
  } else {
    // invoked by VM thread; usually nested VM operation
    assert(t->is_VM_thread(), "must be a VM thread");
    VM_Operation* prev_vm_operation = vm_operation();
    if (prev_vm_operation != NULL) {
      // Check the VM operation allows nested VM operation. This normally not the case, e.g., the compiler
      // does not allow nested scavenges or compiles.
      if (!prev_vm_operation->allow_nested_vm_operations()) {
        fatal("Nested VM operation %s requested by operation %s",
              op->name(), vm_operation()->name());
      }
      op->set_calling_thread(prev_vm_operation->calling_thread(), prev_vm_operation->priority());
    }

    EventMark em("Executing %s VM operation: %s", prev_vm_operation ? "nested" : "", op->name());

    // Release all internal handles after operation is evaluated
    HandleMark hm(t);
    _cur_vm_operation = op;

    if (op->evaluate_at_safepoint() && !SafepointSynchronize::is_at_safepoint()) {
      SafepointSynchronize::begin();
      op->evaluate();
      SafepointSynchronize::end();
    } else {
      op->evaluate();
    }

    // Free memory if needed
    if (op->is_cheap_allocated()) delete op;

    _cur_vm_operation = prev_vm_operation;
  }
}
```