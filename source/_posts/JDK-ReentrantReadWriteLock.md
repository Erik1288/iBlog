---
title: JDK-ReentrantReadWriteLock
date: 2019-11-25 16:35:19
tags:
---

### ReentrantReadWriteLock默认的实现中，怎么保证写操作不被大量的读操作饿死
每次在尝试获取读锁时，会去check下等待队列的队头是不是exclusive(写线程)的线程，如果是的话，当前的读线程先block住，让写线程优先去竞争锁。这样写线程就不会饿死。