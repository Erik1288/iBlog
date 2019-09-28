---
title: JVM-Off-Heap-Memory
date: 2017-12-02 15:05:06
tags: JVM
---


### 问什么需要用堆外内存？
除了速度快，还有什么优点？

### 参考资料
http://www.jianshu.com/p/007052ee3773
full gc是怎么回收堆外内存的？

http://www.cnblogs.com/holoyong/p/7266240.html

Cleaner是PhantomReference的子类，并通过自身的next和prev字段维护的一个双向链表。PhantomReference的作用在于跟踪垃圾回收过程，并不会对对象的垃圾回收过程造成任何的影响。
所以cleaner = Cleaner.create(this, new Deallocator(base, size, cap)); 用于对当前构造的DirectByteBuffer对象的垃圾回收过程进行跟踪。
当DirectByteBuffer对象从pending状态 ——> enqueue状态时，会触发Cleaner的clean()，而Cleaner的clean()的方法会实现通过unsafe对堆外内存的释放。

作者：tomas家的小拨浪鼓
链接：https://www.jianshu.com/p/007052ee3773
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
