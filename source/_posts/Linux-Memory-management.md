---
title: Linux-Memory-management
date: 2018-09-25 21:14:18
tags:
---

https://blog.csdn.net/adaptiver/article/details/7084364


Terms

VSS- Virtual Set Size 虚拟耗用内存（包含共享库占用的内存）
RSS- Resident Set Size 实际使用物理内存（包含共享库占用的内存）
PSS- Proportional Set Size 实际使用的物理内存（比例分配共享库占用的内存）
USS- Unique Set Size 进程独自占用的物理内存（不包含共享库占用的内存）
一般来说内存占用大小有如下规律：VSS >= RSS >= PSS >= USS

