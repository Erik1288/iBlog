---
title: JVM-GC-log
date: 2018-12-03 19:24:49
tags: JVM
---


### 知乎大神
https://www.zhihu.com/question/57722838

```
root     28033  169 73.9 1099400184 48649296 ? Sl   Sep25 169373:13 /bin/java -server -Xms8g -Xmx8g -Xmn4g -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0 -XX:SurvivorRatio=8 -XX:+DisableExplicitGC -XX:+ParallelRefProcEnabled -XX:ConcGCThreads=8 -XX:ParallelGCThreads=8 -XX:+PrintFlagsFinal -XX:+PrintReferenceGC -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC -verbose:gc -Xloggc:/opt/logs/rocketmq/mq_gc_%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintAdaptiveSizePolicy -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m -XX:-OmitStackTraceInFastThrow -XX:+AlwaysPreTouch -XX:-UseBiasedLocking -Djava.ext.dirs=/opt/app/rocketmq/bin/../lib -cp .:/opt/app/rocketmq/bin/../conf: com.alibaba.rocketmq.broker.BrokerStartup -c /opt/app/rocketmq/conf/broker-bb.properties
```

``` RocketMQ G1GC
2018-12-03T19:25:54.072+0800: 5993997.460: Total time for which application threads were stopped: 0.1773895 seconds, Stopping threads took:
0.0001456 seconds
{Heap before GC invocations=221684 (full 0):
 garbage-first heap   total 8388608K, used 5827981K [0x00000005c0000000, 0x00000005c1001000, 0x00000007c0000000)
  region size 16384K, 256 young (4194304K), 12 survivors (196608K)
 Metaspace       used 20710K, capacity 21158K, committed 21376K, reserved 1069056K
  class space    used 1954K, capacity 2110K, committed 2176K, reserved 1048576K
2018-12-03T19:26:06.584+0800: 5994009.973: [GC pause (G1 Evacuation Pause) (young)
Desired survivor size 268435456 bytes, new threshold 15 (max 15)
- age   1:   42091200 bytes,   42091200 total
- age   2:   16620824 bytes,   58712024 total
- age   3:    7993816 bytes,   66705840 total
- age   4:    4589584 bytes,   71295424 total
- age   5:    2678752 bytes,   73974176 total
- age   6:    1463136 bytes,   75437312 total
- age   7:    1306248 bytes,   76743560 total
- age   8:    1120960 bytes,   77864520 total
- age   9:    1029392 bytes,   78893912 total
- age  10:    1034696 bytes,   79928608 total
- age  11:     917248 bytes,   80845856 total
- age  12:     815552 bytes,   81661408 total
- age  13:     768432 bytes,   82429840 total
- age  14:     757120 bytes,   83186960 total
- age  15:     753560 bytes,   83940520 total
 5994009.973: [G1Ergonomics (CSet Construction) start choosing CSet, _pending_cards: 75717, predicted base time: 49.80 ms, remaining time: 150.20 ms, target pause time: 200.00 ms]
 5994009.973: [G1Ergonomics (CSet Construction) add young regions to CSet, eden: 244 regions, survivors: 12 regions, predicted young region time: 96.66 ms]
 5994009.973: [G1Ergonomics (CSet Construction) finish choosing CSet, eden: 244 regions, survivors: 12 regions, old: 0 regions, predicted pause time: 146.47 ms, target pause time: 200.00 ms]
2018-12-03T19:26:06.735+0800: 5994010.123: [SoftReference, 12 refs, 0.0009291 secs]2018-12-03T19:26:06.735+0800: 5994010.124: [WeakReference, 0 refs, 0.0004160 secs]2018-12-03T19:26:06.736+0800: 5994010.125: [FinalReference, 241242 refs, 0.0263976 secs]2018-12-03T19:26:06.762+0800: 5994010.151: [PhantomReference, 0 refs, 18 refs, 0.0009041 secs]2018-12-03T19:26:06.763+0800: 5994010.152: [JNI Weak Reference, 0.0000129 secs], 0.1830687 secs]
   [Parallel Time: 150.0 ms, GC Workers: 8]
      [GC Worker Start (ms): Min: 5994009972.9, Avg: 5994009973.0, Max: 5994009973.1, Diff: 0.1]
      [Ext Root Scanning (ms): Min: 0.3, Avg: 0.5, Max: 1.1, Diff: 0.8, Sum: 3.8]
      [Update RS (ms): Min: 17.0, Avg: 17.7, Max: 17.8, Diff: 0.8, Sum: 141.2]
         [Processed Buffers: Min: 45, Avg: 52.6, Max: 71, Diff: 26, Sum: 421]
      [Scan RS (ms): Min: 52.7, Avg: 53.5, Max: 53.8, Diff: 1.1, Sum: 428.4]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
      [Object Copy (ms): Min: 77.7, Avg: 77.9, Max: 78.9, Diff: 1.2, Sum: 623.2]
      [Termination (ms): Min: 0.0, Avg: 0.2, Max: 0.2, Diff: 0.2, Sum: 1.6]
         [Termination Attempts: Min: 1, Avg: 442.4, Max: 552, Diff: 551, Sum: 3539]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.1, Max: 0.2, Diff: 0.1, Sum: 0.7]
      [GC Worker Total (ms): Min: 149.8, Avg: 149.9, Max: 150.0, Diff: 0.2, Sum: 1198.8]
      [GC Worker End (ms): Min: 5994010122.8, Avg: 5994010122.9, Max: 5994010122.9, Diff: 0.1]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.6 ms]
   [Other: 32.4 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 29.1 ms]
      [Ref Enq: 1.2 ms]
      [Redirty Cards: 0.6 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 1.0 ms]
   [Eden: 3904.0M(3904.0M)->0.0B(3904.0M) Survivors: 192.0M->192.0M Heap: 5691.4M(8192.0M)->1788.4M(8192.0M)]
Heap after GC invocations=221685 (full 0):
 garbage-first heap   total 8388608K, used 1831342K [0x00000005c0000000, 0x00000005c1001000, 0x00000007c0000000)
  region size 16384K, 12 young (196608K), 12 survivors (196608K)
 Metaspace       used 20710K, capacity 21158K, committed 21376K, reserved 1069056K
  class space    used 1954K, capacity 2110K, committed 2176K, reserved 1048576K
}
 [Times: user=0.00 sys=0.00, real=0.18 secs]
```


```
void RuntimeService::record_safepoint_end() {
  HS_PRIVATE_SAFEPOINT_END();

  // Print the time interval for which the app was stopped
  // during the current safepoint operation.
  log_info(safepoint)("Total time for which application threads were stopped: %3.7f seconds, Stopping threads took: %3.7f seconds",
                      last_safepoint_time_sec(), _last_safepoint_sync_time_sec);

  // update the time stamp to begin recording app time
  _app_timer.update();
  if (UsePerfData) {
    _safepoint_time_ticks->inc(_safepoint_timer.ticks_since_update());
  }
}
```