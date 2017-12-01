---
title: Kafka-FAQ
date: 2017-11-17 15:33:16
tags: Kafka
---

Kafka写CommitLog时用了什么锁机制?

sync;lock-free;reentrant lock


### kafka和RocketMQ设计导致得关键性差异——对多队列得支持程度

http://blog.csdn.net/chunlongyu/article/details/54576649


### 学习资料
http://blog.csdn.net/chunlongyu/article/category/6638499

http://blog.csdn.net/chunlongyu/article/details/54407633

http://blog.csdn.net/a417930422/article/category/6086259

http://blog.csdn.net/lizhitao

http://www.cnblogs.com/huxi2b

http://blog.csdn.net/u014393917/article/category/6332828

阿里中间件团队博客 
http://jm.taobao.org/categories/%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6/


第九课. Kafka高性能之道
    9.1 顺序写磁盘
    9.2 零拷贝
    9.3 批处理
    9.4 基于ISR的动态平衡一致性算法