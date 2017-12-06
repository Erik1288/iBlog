---
title: Netty-Buffer-Allocator
date: 2017-11-15 10:05:55
tags: Netty
---

page  - a page is the smallest unit of memory chunk that can be allocated

一般操作系统的内存page都是4kb    既然这么讲，不是可以     byte a = 1; 这样不就是分配了一个byte么???