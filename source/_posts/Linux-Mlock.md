---
title: Linux-Mlock
date: 2018-09-26 10:05:01
tags:
---


### 笨叔叔
mlock(), mlock2(), and mlockall() lock part or all of the calling
process's virtual address space into RAM, preventing that memory from
being paged to the swap area.

这个是mlock的man。说是将某段虚拟内存锁进Ram，防止对应的物理内存swap out。是不是也可以理解为，锁住 vma和pm的 mapping关系，不被删除？

不完全正确
内存 不加入 LRU链表，所以也不会 被 swap out