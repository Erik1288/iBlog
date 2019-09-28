---
title: RocketMQ-Pagecache
date: 2018-08-11 15:46:50
tags:
---

### RocketMQ对


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



```
public void warmMappedFile(FlushDiskType type, int pages) {
    long beginTime = System.currentTimeMillis();
    ByteBuffer byteBuffer = this.mappedByteBuffer.slice();
    int flush = 0;
    long time = System.currentTimeMillis();
    for (int i = 0, j = 0; i < this.fileSize; i += MappedFile.OS_PAGE_SIZE, j++) {
        byteBuffer.put(i, (byte) 0);
        // force flush when flush disk type is sync
        if (type == FlushDiskType.SYNC_FLUSH) {
            if ((i / OS_PAGE_SIZE) - (flush / OS_PAGE_SIZE) >= pages) {
                flush = i;
                mappedByteBuffer.force();
            }
        }

        // prevent gc
        if (j % 1000 == 0) {
            log.info("j={}, costTime={}", j, System.currentTimeMillis() - time);
            time = System.currentTimeMillis();
            try {
                Thread.sleep(0);
            } catch (InterruptedException e) {
                log.error("Interrupted", e);
            }
        }
    }

    // force flush when prepare load finished
    if (type == FlushDiskType.SYNC_FLUSH) {
        log.info("mapped file warm-up done, force to disk, mappedFile={}, costTime={}",
            this.getFileName(), System.currentTimeMillis() - beginTime);
        mappedByteBuffer.force();
    }
    log.info("mapped file warm-up done. mappedFile={}, costTime={}", this.getFileName(),
        System.currentTimeMillis() - beginTime);

    this.mlock();
}

public void mlock() {
        final long beginTime = System.currentTimeMillis();
        final long address = ((DirectBuffer) (this.mappedByteBuffer)).address();
        Pointer pointer = new Pointer(address);
        {
            int ret = LibC.INSTANCE.mlock(pointer, new NativeLong(this.fileSize));
            log.info("mlock {} {} {} ret = {} time consuming = {}", address, this.fileName, this.fileSize, ret, System.currentTimeMillis() - beginTime);
        }

        {
            int ret = LibC.INSTANCE.madvise(pointer, new NativeLong(this.fileSize), LibC.MADV_WILLNEED);
            log.info("madvise {} {} {} ret = {} time consuming = {}", address, this.fileName, this.fileSize, ret, System.currentTimeMillis() - beginTime);
        }
    }
```
这个写入一个随机值，告诉内核，强制让内核发生缺页中断，去做虚拟内存和物理内存的mapping。注意，这个是给预创建的Commit Log。即使做了mapping，这部分的内存物理内存还是会变冷（LRU），还是有可能被内核回收走，干别的事情。这时要调用 mlock，告诉内核，这部分虚拟内存mapping的物理内存，千万别swap out 到swap区域，要持续放入物理内存中。madvise 传入will_need是高速操作系统，这块内存我等下就要使用，即使我暂时不用，也不要让它变冷，而被回收走。



在 2018年4月8日 上午10:07，Zhanhui Li <lizhanhui@apache.org>写道：

> writeBuffer 是预分配的anonymous pages, 并且mlock到物理内存了. 往里面写消息,
不会因为内存不足回收带来延迟.
> mmap的文件page cache, 在物理内存分配过快, background reclaim速度跟不上的时候,
线程会被block住,进行direct reclaim, 带来延迟.
>
> 可参考:  https://events.static.linuxfound.org/sites/events/
> files/lcjp13_moriya.pdf