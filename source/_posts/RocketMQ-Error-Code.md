---
title: RocketMQ-Error-Code
date: 2018-11-30 11:39:21
tags:
---


### PCBUSY_CLEAN_QUEUE
```
org.apache.rocketmq.broker.latency.BrokerFastFailure
```



### TIMEOUT_CLEAN_QUEUE
```
org.apache.rocketmq.broker.latency.BrokerFastFailure
RocketMQ Broker作为一台服务器，会并发接受大量的请求(SendMessageRequest, PullMessageRequest, HeartbeatRequest)。但是如果服务器由于某些原因无法及时处理请求，需要FastFailure，让客户端及时作出反应。
服务器处理慢的原因：
1. Page Cache busy，也就是刷盘的时间超过osPageCacheBusyTimeOutMills(1000ms by default)，这时Broker会把SendMessage线程池里面所有排队的请求都直接返回
[PCBUSY_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: %sms, size of queue: %d]
@Override
public boolean isOSPageCacheBusy() {
    long begin = this.getCommitLog().getBeginTimeInLock();
    long diff = this.systemClock.now() - begin;

    return diff < 10000000
            && diff > this.messageStoreConfig.getOsPageCacheBusyTimeOutMills();
}
2.



2018-12-03 18:41:45 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 204 ms, size of queue: 0
2018-12-03 18:47:48 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 204 ms, size of queue: 1
2018-12-03 18:47:48 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 204 ms, size of queue: 0
2018-12-03 18:48:47 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 204 ms, size of queue: 0
2018-12-03 18:49:10 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 205 ms, size of queue: 0
2018-12-03 18:49:56 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 258 ms, size of queue: 1
2018-12-03 18:49:56 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 258 ms, size of queue: 0
2018-12-03 18:51:29 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 206 ms, size of queue: 0
2018-12-03 18:53:51 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 200 ms, size of queue: 24
2018-12-03 18:53:51 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 200 ms, size of queue: 23
2018-12-03 18:53:51 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 200 ms, size of queue: 22
2018-12-03 18:53:51 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 200 ms, size of queue: 21
2018-12-03 18:53:51 WARN BrokerFastFailureScheduledThread1 - [TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: 200 ms, size of queue: 20

```


Consumer限流
```
if (cachedMessageCount > this.defaultMQPushConsumer.getPullThresholdForQueue()) {
    this.executePullRequestLater(pullRequest, PULL_TIME_DELAY_MILLS_WHEN_FLOW_CONTROL);
    if ((queueFlowControlTimes++ % 1000) == 0) {
        log.warn(
            "the cached message count exceeds the threshold {}, so do flow control, minOffset={}, maxOffset={}, count={}, size={} MiB, pullRequest={}, flowControlTimes={}",
            this.defaultMQPushConsumer.getPullThresholdForQueue(), processQueue.getMsgTreeMap().firstKey(), processQueue.getMsgTreeMap().lastKey(), cachedMessageCount, cachedMessageSizeInMiB, pullRequest, queueFlowControlTimes);
    }
    return;
}
2018-12-18 12:29:41,041 WARN RocketmqClient - the cached message count exceeds the threshold 1000, so do flow control, minOffset=68, maxOffset=1091, count=1024, size=0 MiB, pullRequest=PullRequest [consumerGroup=eric_cg, messageQueue=MessageQueue [topic=EricTpc, brokerName=broker-a, queueId=2], nextOffset=1092], flowControlTimes=711001
```




### 刷盘慢问题，看日志是文件预热的时候，对SSD压力过大
```
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=251000, costTime=5
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=252000, costTime=5
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=253000, costTime=4
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=254000, costTime=5
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=255000, costTime=5
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=256000, costTime=4
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=257000, costTime=5
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=258000, costTime=5
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=259000, costTime=5
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=260000, costTime=5
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=261000, costTime=5
2019-03-22 10:34:39 INFO AllocateMappedFileService - j=262000, costTime=5
2019-03-22 10:34:39 INFO AllocateMappedFileService - mapped file warm-up done. mappedFile=/opt/data/rocketmq/store/commitlog/00000037095632535552, costTime=4450
2019-03-22 10:34:39 INFO AllocateMappedFileService - mlock 139888250814464 /opt/data/rocketmq/store/commitlog/00000037095632535552 1073741824 ret = 0 time consuming = 161
2019-03-22 10:34:39 INFO AllocateMappedFileService - madvise 139888250814464 /opt/data/rocketmq/store/commitlog/00000037095632535552 1073741824 ret = 0 time consuming = 161
2019-03-22 10:34:41 INFO StoreScheduledThread1 - munlock 139890398298112 /opt/data/rocketmq/store/commitlog/00000037093485051904 1073741824 ret = 0 time consuming = 123

sar -B
         AM  pgpgin/s pgpgout/s   fault/s  majflt/s  pgfree/s pgscank/s pgscand/s pgsteal/s    %vmeff
10:33:01 AM      2.27   2490.55    403.91      0.05  15716.03      0.00      0.00      0.00      0.00
10:34:01 AM      0.00   2115.41    532.05      0.00  15597.60      0.00      0.00      0.00      0.00
10:35:01 AM      0.80  22780.21   5033.77      0.03  22221.77   3186.27   6828.45   6035.93     60.27
10:36:01 AM      0.00   2487.04    406.26      0.00  17695.18      0.00    887.67    887.67    100.00
10:37:01 AM      0.93   2262.00    481.78      0.02  17252.16      0.00    812.66    812.66    100.00

2019-03-22 10:41:42 INFO AllocateMappedFileService - j=242000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=243000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=244000, costTime=5
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=245000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=246000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=247000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=248000, costTime=5
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=249000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=250000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=251000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=252000, costTime=5
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=253000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=254000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=255000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=256000, costTime=5
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=257000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=258000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=259000, costTime=5
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=260000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=261000, costTime=4
2019-03-22 10:41:43 INFO AllocateMappedFileService - j=262000, costTime=3
2019-03-22 10:41:43 INFO AllocateMappedFileService - mapped file warm-up done. mappedFile=/opt/data/rocketmq/store/commitlog/00000037096706277376, costTime=1222
2019-03-22 10:41:43 INFO AllocateMappedFileService - mlock 139887177072640 /opt/data/rocketmq/store/commitlog/00000037096706277376 1073741824 ret = 0 time consuming = 170
2019-03-22 10:41:43 INFO AllocateMappedFileService - madvise 139887177072640 /opt/data/rocketmq/store/commitlog/00000037096706277376 1073741824 ret = 0 time consuming = 170
2019-03-22 10:41:43 INFO FlushRealTimeService - Flush data to disk costs 2074 ms
2019-03-22 10:41:47 INFO StoreScheduledThread1 - munlock 139889324556288 /opt/data/rocketmq/store/commitlog/00000037094558793728 1073741824 ret = 0 time consuming = 116


sar -B
         AM  pgpgin/s pgpgout/s   fault/s  majflt/s  pgfree/s pgscank/s pgscand/s pgsteal/s    %vmeff
10:40:01 AM      2.53   2404.46    536.20      0.05  13776.71      0.00      0.00      0.00      0.00
10:41:01 AM      0.00   6429.72    426.92      0.00  18859.91      0.00      0.00      0.00      0.00
10:42:01 AM      0.60  21695.09   4842.20      0.03  16396.72      0.00      0.00      0.00      0.00
10:43:01 AM      0.93   2507.08    476.69      0.02  14196.68      0.00      0.00      0.00      0.00
10:44:02 AM      2.40   2343.42    404.17      0.05  14700.77      0.00      0.00      0.00      0.00

```
