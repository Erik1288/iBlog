---
title: JDK-AbstractQueuedSynchronizer
date: 2019-11-25 17:07:21
tags:
---


### AQS内部三个基础组件：
1. Unsafe.cas()
2. LockSupport.park()/unpark()
3. CLH Queue: sync queue & condition queue