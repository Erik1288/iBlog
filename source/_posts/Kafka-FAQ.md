---
title: Kafka-FAQ
date: 2017-11-17 15:33:16
tags: Kafka
---

Kafka写CommitLog时用了什么锁机制?

sync;lock-free;reentrant lock



第九课. Kafka高性能之道
    9.1 顺序写磁盘
    9.2 零拷贝
    9.3 批处理
    9.4 基于ISR的动态平衡一致性算法