---
title: Sofa-Jraft
date: 2019-04-02 17:03:23
tags:
---


https://tech.antfin.com/activities/382/review/712
ppt
https://mp.weixin.qq.com/s/zDusnG6WJGP0EX8UmbqtxQ


### 源码分析文章
https://zhuanlan.zhihu.com/p/66355477


### 什么是线性一致读？


### 在写WAL时，Log的Key是什么？单纯的LogIndex应该是没法满足需求的。
这个问题有没有问对？如果Index是严格单调递增的呢？
LogEntry
/** log id with index/term */
    private LogId                id    = new LogId(0, 0);


Batch: 我们知道互联网两大优化法宝便是 cache 和 batch，JRaft 在 batch 上花了较大心思，整个链路几乎都是 batch 的，依靠 disruptor 的 MPSC 模型批量消费，对整体性能有着极大的提升，包括但不限于：
批量提交 task
批量网络发送
本地 IO batch 写入
要保证日志不丢，一般每条 log entry 都要进行 fsync 同步刷盘，比较耗时，JRaft 中做了合并写入的优化
批量应用到状态机 需要说明的是，虽然 JRaft 中大量使用了 batch 技巧，但对单个请求的延时并无任何影响，JRaft 中不会对请求做延时的攒批处理

// 好好分析下这个Batch模型
@Override
public void onEvent(LogEntryAndClosure event, long sequence, boolean endOfBatch) throws Exception {
    if (event.shutdownLatch != null) {
        if (!tasks.isEmpty()) {
            executeApplyingTasks(tasks);
        }
        GLOBAL_NUM_NODES.decrementAndGet();
        event.shutdownLatch.countDown();
        return;
    }

    tasks.add(event);
    if (tasks.size() >= raftOptions.getApplyBatch() || endOfBatch) {
        executeApplyingTasks(tasks);
        tasks.clear();
    }
}


########### 重要代码执行过程
## PreVote和Vote到BecomeLeader的过程

## Leader收到Client请求，将日志作为WAL持久化在本地。
## Leader在持久化成功后（可能是并发进行的），
## Leader发送AppendEntries后，收到了大部分的确认，开始应用状态机。
## Leader成功应用状态机，将commitIndex通过心跳传递给Follow，Follow开始应用状态机。

## Follow什么时候需要被安装Snapshot，怎么安装？

## 如果Leader挂了，可能有一些废弃的LocalLog，怎么Trancate掉？
