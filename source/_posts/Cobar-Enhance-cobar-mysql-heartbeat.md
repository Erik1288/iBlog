---
title: Cobar-Enhance-cobar-mysql-heartbeat
date: 2017-10-27 14:37:10
tags:
---


### Cobar与MySQL心跳实现

Cobar的心跳是用原生NIO写的，相对于用Netty来发送心跳，复杂度不在一个量级上。

### 网络抖动

Cobar会向MySQL发送`select user()`的心跳，Cobar会在以下两种情况下进行数据源切换
* Cobar向MySQL发送心跳数据失败，界定为Error
* Cobar向MySQL发送心跳数据，但30秒（默认）内没有收到MySQL回复，界定为Timeout

在阿里云VPC的环境中，网络抖动又是家常便饭，一旦网络出现抖动，Cobar立刻把对应dataNode的数据源从Master切到Slave或者从Slave切回Master，于是，慢SQL出现了。这时都意识到，这个数据源切换的策略不靠谱，需要重新设计。

不能一发现情况一，就认定是Error，网络抖动很有可能在几秒钟之内恢复回来，这里采用的策略是，发现情况一，再试3次，如果第3次还失败，等待n秒后再试一次，如果还发送心跳失败，那就是Error，需要切换数据源。

### 过滤抖动策略