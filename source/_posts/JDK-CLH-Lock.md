---
title: JDK-CLH-Lock
date: 2019-05-27 22:10:42
tags:
---


https://www.cnblogs.com/llkmst/p/4895478.html
CLH(Craig, Landin, and Hagersten  locks): 是一个自旋锁，能确保无饥饿性，提供先来先服务的公平性。
CLH锁也是一种基于链表的可扩展、高性能、公平的自旋锁，申请线程只在本地变量上自旋，它不断轮询前驱的状态，如果发现前驱释放了锁就结束自旋。