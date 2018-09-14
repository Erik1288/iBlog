---
title: Linux-Virtual-memory
date: 2018-09-14 19:27:23
tags:
---



https://blog.csdn.net/tenfyguo/article/details/8575473
```
一，不是 malloc 后就马上占用实际内存，而是第一次使用时（如对内存赋值，memset等操作）发现虚存对应的物理页面未分配，产生缺页中断，才真正分配物理页面，
同时更新进程页面的映射关系。但由于每个物理内存页面大小是 4k ，不管 memset其中的1k还是5k 、7k ，实际占用物理内存总是 4k 的倍数。所以 RSS 的增量总是 4k 的倍数。
```