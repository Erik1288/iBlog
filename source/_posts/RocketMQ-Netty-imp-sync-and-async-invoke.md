---
title: RocketMQ——Netty实现的远程同步与异步调用（二）
date: 2017-08-21 12:36:42
tags: RocketMQ
---

### Netty IO框架
netty是一个异步IO框架，异步API最大的特点就是基于事件，netty当然也不例外。

### Remote同步调用
![你想输入的替代文字](RocketMQ-Netty-imp-sync-and-async-invoke/invokeSync.png)

### Remote异步调用
异步调用不会使Caller线程等待，理论上可以在短时间内不限次数得调用，这将对系统造成非常大压力，所以在异步调用设计中引入了限流机制
![你想输入的替代文字](RocketMQ-Netty-imp-sync-and-async-invoke/invokeAsync.png)

###