---
title: JDK-ThreadPoolExecutor
date: 2019-11-27 17:27:56
tags:
---



### 一个Worker代表着一个线程，如果某个Worker长时间没有从队列中取到任务，为了节省系统资源，会不会把线程给关闭掉？