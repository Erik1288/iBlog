---
title: JDK-ReentrantLock
date: 2019-11-26 16:17:49
tags:
---

### 为什么用了ReentrantLock后，在lock()和unlock()之间，不用担心指令重排序的？

### 为什么用CAS加锁的{}中，需要对数据加volatile限定词？