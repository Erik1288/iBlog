---
title: RocketMQ-TransientStorePool
date: 2018-11-06 18:02:28
tags:
---

https://processon.com/diagraming/5be2bdede4b0ad314e815528

操作系统内部有很多非常优秀的设计，但是，有时候这些优秀的设计并不能服务于那些需要极致性能的软件，所以有时我们需要用它的思想来设计我们自己的软件。



``` 在64G内存的机器上刷盘还会出现大量毛刺，会不会是切换CommitLog时，没有做warmup造成的？
2018-10-28 11:23:17 WARN SendMessageThread_1 - [NOTIFYME]putMessage in lock cost time(ms)=1408, bodyLength=2255 AppendMessageResult=AppendMessageResult{status=PUT_OK, wroteOffset=1253026167641, wroteBytes=2477, msgId='0A0A41C900002A9F00000123BE2DFB59', storeTimestamp=1540696996021, logicsOffset=21187262, pagecacheRT=1408, msgNum=1}
2018-10-27 19:54:56 WARN SendMessageThread_1 - [NOTIFYME]putMessage in lock cost time(ms)=5136, bodyLength=1815 AppendMessageResult=AppendMessageResult{status=PUT_OK, wroteOffset=1192894396447, wroteBytes=2029, msgId='0A0A41C900002A9F00000115BE0BF81F', storeTimestamp=1540641291355, logicsOffset=18326688, pagecacheRT=5136, msgNum=1}
2018-10-27 18:00:45 WARN SendMessageThread_1 - [NOTIFYME]putMessage in lock cost time(ms)=3307, bodyLength=3226 AppendMessageResult=AppendMessageResult{status=PUT_OK, wroteOffset=1148235151172, wroteBytes=3454, msgId='0A0A41C900002A9F0000010B5825F744', storeTimestamp=1540634442011, logicsOffset=26631653, pagecacheRT=3306, msgNum=1}
2018-10-27 13:12:16 WARN SendMessageThread_1 - [NOTIFYME]putMessage in lock cost time(ms)=4586, bodyLength=2814 AppendMessageResult=AppendMessageResult{status=PUT_OK, wroteOffset=1103117022509, wroteBytes=3037, msgId='0A0A41C900002A9F00000100D6E5F52D', storeTimestamp=1540617131925, logicsOffset=14710837, pagecacheRT=4586, msgNum=1}
```



先看定义
```
public class MappedFile extends ReferenceResource {
    protected FileChannel fileChannel;
    /**
     * Message will put to here first, and then reput to FileChannel if writeBuffer is not null.
     */
    protected ByteBuffer writeBuffer = null;
    protected TransientStorePool transientStorePool = null;
}
```

```
/**
 * It's a heavy init method.
 */
public void init() {
    for (int i = 0; i < poolSize; i++) {
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(fileSize);

        final long address = ((DirectBuffer) byteBuffer).address();
        Pointer pointer = new Pointer(address);
        LibC.INSTANCE.mlock(pointer, new NativeLong(fileSize));

        availableBuffers.offer(byteBuffer);
    }
}
```

AllocateMappedFileService#mmapOperation


先梳理两个重要的对象，flushCommitLogService和commitLogService，它们都是类型都是FlushCommitLogService。
```
private final FlushCommitLogService flushCommitLogService;

//If TransientStorePool enabled, we must flush message to FileChannel at fixed periods
private final FlushCommitLogService commitLogService;

public CommitLog(final DefaultMessageStore defaultMessageStore) {
    this.mappedFileQueue = new MappedFileQueue(defaultMessageStore.getMessageStoreConfig().getStorePathCommitLog(),
        defaultMessageStore.getMessageStoreConfig().getMapedFileSizeCommitLog(), defaultMessageStore.getAllocateMappedFileService());
    this.defaultMessageStore = defaultMessageStore;

    if (FlushDiskType.SYNC_FLUSH == defaultMessageStore.getMessageStoreConfig().getFlushDiskType()) {
        this.flushCommitLogService = new GroupCommitService();
    } else {
        this.flushCommitLogService = new FlushRealTimeService();
    }

    this.commitLogService = new CommitRealTimeService();

    this.appendMessageCallback = new DefaultAppendMessageCallback(defaultMessageStore.getMessageStoreConfig().getMaxMessageSize());
    batchEncoderThreadLocal = new ThreadLocal<MessageExtBatchEncoder>() {
        @Override
        protected MessageExtBatchEncoder initialValue() {
            return new MessageExtBatchEncoder(defaultMessageStore.getMessageStoreConfig().getMaxMessageSize());
        }
    };
    this.putMessageLock = defaultMessageStore.getMessageStoreConfig().isUseReentrantLockWhenPutMessage() ? new PutMessageReentrantLock() : new PutMessageSpinLock();

}
```




### 刷盘实现
```
class CommitRealTimeService extends FlushCommitLogService {

    private long lastCommitTimestamp = 0;

    @Override
    public String getServiceName() {
        return CommitRealTimeService.class.getSimpleName();
    }

    @Override
    public void run() {
        CommitLog.log.info(this.getServiceName() + " service started");
        while (!this.isStopped()) {
            int interval = CommitLog.this.defaultMessageStore.getMessageStoreConfig().getCommitIntervalCommitLog();

            int commitDataLeastPages = CommitLog.this.defaultMessageStore.getMessageStoreConfig().getCommitCommitLogLeastPages();

            int commitDataThoroughInterval =
                CommitLog.this.defaultMessageStore.getMessageStoreConfig().getCommitCommitLogThoroughInterval();

            long begin = System.currentTimeMillis();
            if (begin >= (this.lastCommitTimestamp + commitDataThoroughInterval)) {
                this.lastCommitTimestamp = begin;
                commitDataLeastPages = 0;
            }

            try {
                boolean result = CommitLog.this.mappedFileQueue.commit(commitDataLeastPages);
                long end = System.currentTimeMillis();
                if (!result) {
                    this.lastCommitTimestamp = end; // result = false means some data committed.
                    //now wake up flush thread.
                    flushCommitLogService.wakeup();
                }

                if (end - begin > 500) {
                    log.info("Commit data to file costs {} ms", end - begin);
                }
                this.waitForRunning(interval);
            } catch (Throwable e) {
                CommitLog.log.error(this.getServiceName() + " service has exception. ", e);
            }
        }

        boolean result = false;
        for (int i = 0; i < RETRY_TIMES_OVER && !result; i++) {
            result = CommitLog.this.mappedFileQueue.commit(0);
            CommitLog.log.info(this.getServiceName() + " service shutdown, retry " + (i + 1) + " times " + (result ? "OK" : "Not OK"));
        }
        CommitLog.log.info(this.getServiceName() + " service end");
    }
}
```

```

```