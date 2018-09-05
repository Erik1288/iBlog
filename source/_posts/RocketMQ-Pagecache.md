---
title: RocketMQ-Pagecache
date: 2018-08-11 15:46:50
tags:
---


```
man free
DESCRIPTION
    cache  Memory used by the page cache and slabs (Cached and Slab in /proc/meminfo)
```

### Linux文件读写机制及优化方式
https://blog.csdn.net/littlewhite1989/article/details/52583879

pagecache是RocketMQ高性能实现上使用的一个利器。虽然源码中没有对pagecache的做过多解释，但是作为RocketMQ的研究者，必须清楚得认识到pagecache对于性能的影响有多么巨大。本文借用一些工具帮助我们更好理解RocketMQ是如何在设计上就利用上pagecache。

首先推荐一款工具pcstat(https://github.com/tobert/pcstat)，它可以直观得显示出指定文件是否被cache，甚至可以知道cache的比例是多少。
```
[root@mq003 bin]# ./pcstat /opt/data/rocketmq/store/commitlog/*
|---------------------------------------------------------+----------------+------------+-----------+---------|
| Name                                                    | Size           | Pages      | Cached    | Percent |
|---------------------------------------------------------+----------------+------------+-----------+---------|
| /opt/data/rocketmq/store/commitlog/00000000303868936192 | 1073741824     | 262144     | 40        | 000.015 |
| /opt/data/rocketmq/store/commitlog/00000000304942678016 | 1073741824     | 262144     | 18        | 000.007 |
| /opt/data/rocketmq/store/commitlog/00000000306016419840 | 1073741824     | 262144     | 1         | 000.000 |
| /opt/data/rocketmq/store/commitlog/00000000307090161664 | 1073741824     | 262144     | 1         | 000.000 |
| /opt/data/rocketmq/store/commitlog/00000000308163903488 | 1073741824     | 262144     | 1         | 000.000 |
| /opt/data/rocketmq/store/commitlog/00000000309237645312 | 1073741824     | 262144     | 4         | 000.002 |
| /opt/data/rocketmq/store/commitlog/00000000310311387136 | 1073741824     | 262144     | 0         | 000.000 |
| /opt/data/rocketmq/store/commitlog/00000000311385128960 | 1073741824     | 262144     | 0         | 000.000 |
| /opt/data/rocketmq/store/commitlog/00000000312458870784 | 1073741824     | 262144     | 0         | 000.000 |
| /opt/data/rocketmq/store/commitlog/00000000313532612608 | 1073741824     | 262144     | 16        | 000.006 |
| /opt/data/rocketmq/store/commitlog/00000000314606354432 | 1073741824     | 262144     | 16        | 000.006 |
| /opt/data/rocketmq/store/commitlog/00000000315680096256 | 1073741824     | 262144     | 2         | 000.001 |
| /opt/data/rocketmq/store/commitlog/00000000316753838080 | 1073741824     | 262144     | 0         | 000.000 |
| /opt/data/rocketmq/store/commitlog/00000000317827579904 | 1073741824     | 262144     | 2         | 000.001 |
| /opt/data/rocketmq/store/commitlog/00000000318901321728 | 1073741824     | 262144     | 11        | 000.004 |
| /opt/data/rocketmq/store/commitlog/00000000319975063552 | 1073741824     | 262144     | 5         | 000.002 |
| /opt/data/rocketmq/store/commitlog/00000000321048805376 | 1073741824     | 262144     | 40        | 000.015 |
| /opt/data/rocketmq/store/commitlog/00000000322122547200 | 1073741824     | 262144     | 14        | 000.005 |
| /opt/data/rocketmq/store/commitlog/00000000323196289024 | 1073741824     | 262144     | 17        | 000.006 |
| /opt/data/rocketmq/store/commitlog/00000000324270030848 | 1073741824     | 262144     | 18        | 000.007 |
| /opt/data/rocketmq/store/commitlog/00000000325343772672 | 1073741824     | 262144     | 17        | 000.006 |
| /opt/data/rocketmq/store/commitlog/00000000326417514496 | 1073741824     | 262144     | 13        | 000.005 |
| /opt/data/rocketmq/store/commitlog/00000000327491256320 | 1073741824     | 262144     | 5         | 000.002 |
| /opt/data/rocketmq/store/commitlog/00000000328564998144 | 1073741824     | 262144     | 13        | 000.005 |
| /opt/data/rocketmq/store/commitlog/00000000329638739968 | 1073741824     | 262144     | 16        | 000.006 |
| /opt/data/rocketmq/store/commitlog/00000000330712481792 | 1073741824     | 262144     | 0         | 000.000 |
| /opt/data/rocketmq/store/commitlog/00000000331786223616 | 1073741824     | 262144     | 0         | 000.000 |
| /opt/data/rocketmq/store/commitlog/00000000332859965440 | 1073741824     | 262144     | 2         | 000.001 |
| /opt/data/rocketmq/store/commitlog/00000000333933707264 | 1073741824     | 262144     | 36        | 000.014 |
| /opt/data/rocketmq/store/commitlog/00000000335007449088 | 1073741824     | 262144     | 29313     | 011.182 |
| /opt/data/rocketmq/store/commitlog/00000000336081190912 | 1073741824     | 262144     | 0         | 000.000 |
|---------------------------------------------------------+----------------+------------+-----------+---------|
```
可以看到`/opt/data/rocketmq/store/commitlog/00000000335007449088`有11.182%都被cache了
