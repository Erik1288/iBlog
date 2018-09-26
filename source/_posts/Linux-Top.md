---
title: Linux-Top
date: 2018-09-26 12:42:06
tags:
---

用top -H定位RocketMQ最耗CPU的线程在做什么事情
```
top -H

按住shift+p，让线程按照CPU从大到小排列

top - 12:44:35 up 14 days, 19:56,  3 users,  load average: 0.22, 0.25, 0.31
Threads: 386 total,   3 running, 383 sleeping,   0 stopped,   0 zombie
%Cpu(s): 12.4 us,  6.7 sy,  0.0 ni, 77.4 id,  2.0 wa,  0.0 hi,  1.1 si,  0.4 st
KiB Mem : 90.6/3881920  [||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||          ]
KiB Swap: 11.2/1048572  [|||||||||||                                                                                         ]

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                                            
28690 root      20   0 14.263g 2.862g 207296 S 10.0 77.3   0:12.49 java                                                                                                                               
28598 root      20   0 14.263g 2.862g 207296 S  7.3 77.3   0:26.65 java                                                                                                                               
28589 root      20   0 14.263g 2.862g 207296 S  7.0 77.3   0:58.14 java                                                                                                                               
27914 root      20   0 14.263g 2.862g 207296 S  5.0 77.3   1:18.72 java                                                                                                                               
27874 root      20   0 14.263g 2.862g 207296 R  4.7 77.3   2:04.93 java                                                                                                                               
27887 root      20   0 14.263g 2.862g 207296 S  4.7 77.3   2:27.30 java                                                                                                                               
27872 root      20   0 14.263g 2.862g 207296 S  1.7 77.3   0:13.04 java                                                                                                                               
  259 root      20   0       0      0      0 S  1.0  0.0   2:09.55 jbd2/vda1-8                                                                                                                        
27896 root      20   0 14.263g 2.862g 207296 S  0.7 77.3   1:48.26 java                                                                                                                               
27924 root      20   0 14.263g 2.862g 207296 S  0.7 77.3   0:03.36 java                                                                                                                               
28940 root      20   0 2846000 216716   6820 S  0.7  5.6   0:16.50 java                                                                                                                               
28951 root      20   0 2846000 216716   6820 S  0.7  5.6   0:44.03 java                                                                                                                               
    6 root      20   0       0      0      0 S  0.3  0.0   1:04.50 kworker/u4:0                                                                                                                       
  391 root       0 -20       0      0      0 S  0.3  0.0   0:24.38 kworker/0:1H  
  

线程号为28690线程占用了最多的CPU

printf "%x\n" 28690
7012

用该线程号去java栈中找，

[root@mq004 rocketmqlogs]# jstack 27782 | grep -A29 7012
"NettyServerCodecThread_4" #62 prio=5 os_prio=0 tid=0x00007f13cc019510 nid=0x7012 waiting on condition [0x00007f11c9c0d000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007274f1b30> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
	at java.util.concurrent.LinkedBlockingQueue.poll(LinkedBlockingQueue.java:467)
	at io.netty.util.concurrent.SingleThreadEventExecutor.takeTask(SingleThreadEventExecutor.java:269)
	at io.netty.util.concurrent.DefaultEventExecutor.run(DefaultEventExecutor.java:39)
	at io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:140)
	at java.lang.Thread.run(Thread.java:748)

"ClientManageThread_15" #165 prio=5 os_prio=0 tid=0x00007f139c802840 nid=0x6ff6 waiting on condition [0x00007f11c9d0e000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007260004c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
	at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
	at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)

"NettyServerCodecThread_3" #61 prio=5 os_prio=0 tid=0x00007f13b802fa50 nid=0x6fd1 waiting on condition [0x00007f11c9e0f000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007274f1c08> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
	
	
发现NettyServerCodecThread_4占用的CPU比较多，从名字上也能看出，这个线程是用来codec的。
```

