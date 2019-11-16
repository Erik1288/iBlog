---
title: Flink-FAQ
date: 2018-12-28 07:29:22
tags:
---

### 超高质量文章
https://mlog.club/articles/tag/2178

### 很多好问题
https://mlog.club/article/40687
https://www.aboutyun.com/thread-27738-1-1.html

### 
https://blog.csdn.net/fct2001140269/article/details/87357949

### 现在要处理一批数据，KeyBy(entityId)后，发现某几个entityId数据量特别大，可能是其它的好几倍，怎么处理？

### 当Flink的Checkpoint非常大时，怎么样进行低延时保存？
https://www.ververica.com/blog/managing-large-state-apache-flink-incremental-checkpointing-overview

### 这个例子就没有并发问题么？
https://training.ververica.com/exercises/rideEnrichment-flatmap.html


### Flink与Kafka融合时，Kafka的游标时怎么管理的？


### Sliding Window的数据复制
Sliding Windows Make Copies
Sliding window assigners can create lots of window objects, and will copy each event into every relevant window. For example, if you have sliding windows every 15 minutes that are 24-hours in length, each event will be copied into 4 * 24 = 96 windows.

### 
msg1,t1
msg2,t2
msg3,t3
msg4,t4
...
以这个顺序进入Flink
msg1,t1
msg2,t2
msg4,t4
msg5,t5
msg6,t6
msg7,t7
...
10条
...
msg3,t3
### 有一个作业是去统计一个超级大的文件的单词出现的次数，这个文件需要被处理整整一个月，如果中间Streaming作业停掉了，那后续应该怎么恢复才能达到exactly once？因为这个作业的源没有像Kafka那样有下标。

### Flink的slot指的是什么？和线程是什么关系？
按照我现在的理解，10个Slot并不是说有10个线程在执行，而是10个并行线。

source -> map -> aggregation -> sink

### KeyBy修改了并行度后，并行线会发生变化么？

###

### Flink是怎么实现Exactly once？和Kafka有什么区别？
https://mp.weixin.qq.com/s/4EtkNns-KAzEqL3GRMRLAg
StreamingFileSink

### 一个flink的并行任务，对于每一个taskSlot，它保存的是自己Keyed到的State，如果修改了并行度，State怎么变化？

### 如果Kafka的Topic有10个Partition，那Source的时候，设置多少的并行度比较合适？

### 如果已知Kafka的Topic已经按照Entity做了Partition，这个时候Flink的Source是和这个Topic并行的么？