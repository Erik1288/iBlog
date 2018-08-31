---
title: RocketMQ-Persistence
date: 2018-08-31 19:14:24
tags:
---


### 参考MySQL Redo log刷盘机制
https://www.tuicool.com/articles/Y7vUBz


```
innodb_flush_log_at_trx_commit

Controls the balance between strict ACID compliance for commit operations and higher performance that is possible when commit-related I/O operations are rearranged and done in batches. You can achieve better performance by changing the default value but then you can lose transactions in a crash.

The default setting of 1 is required for full ACID compliance. Logs are written and flushed to disk at each transaction commit.

With a setting of 0, logs are written and flushed to disk once per second. Transactions for which logs have not been flushed can be lost in a crash.

With a setting of 2, logs are written after each transaction commit and flushed to disk once per second. Transactions for which logs have not been flushed can be lost in a crash.
```