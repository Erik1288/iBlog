---
title: Pulsar-BrokerCache
date: 2019-06-13 18:59:09
tags:
---



### 写缓存
```
insert(EntryImpl):87, EntryCacheImpl (org.apache.bookkeeper.mledger.impl), EntryCacheImpl.java
safeRun():156, OpAddEntry (org.apache.bookkeeper.mledger.impl), OpAddEntry.java
run():36, SafeRunnable (org.apache.bookkeeper.common.util), SafeRunnable.java
runWorker(ThreadPoolExecutor$Worker):1149, ThreadPoolExecutor (java.util.concurrent), ThreadPoolExecutor.java
run():624, ThreadPoolExecutor$Worker (java.util.concurrent), ThreadPoolExecutor.java
run():30, FastThreadLocalRunnable (io.netty.util.concurrent), FastThreadLocalRunnable.java
run():748, Thread (java.lang), Thread.java
```

### 读缓存
```
getRange(Comparable, Comparable):109, RangeCache (org.apache.bookkeeper.mledger.util), RangeCache.java
asyncReadEntry0(ReadHandle, long, long, boolean, AsyncCallbacks$ReadEntriesCallback, Object):261, EntryCacheImpl (org.apache.bookkeeper.mledger.impl), EntryCacheImpl.java
asyncReadEntry(ReadHandle, long, long, boolean, AsyncCallbacks$ReadEntriesCallback, Object):238, EntryCacheImpl (org.apache.bookkeeper.mledger.impl), EntryCacheImpl.java
asyncReadEntry(ReadHandle, long, long, boolean, OpReadEntry, Object):1553, ManagedLedgerImpl (org.apache.bookkeeper.mledger.impl), ManagedLedgerImpl.java
internalReadFromLedger(ReadHandle, OpReadEntry):1527, ManagedLedgerImpl (org.apache.bookkeeper.mledger.impl), ManagedLedgerImpl.java
asyncReadEntries(OpReadEntry):1380, ManagedLedgerImpl (org.apache.bookkeeper.mledger.impl), ManagedLedgerImpl.java
asyncReadEntries(int, AsyncCallbacks$ReadEntriesCallback, Object):476, ManagedCursorImpl (org.apache.bookkeeper.mledger.impl), ManagedCursorImpl.java
asyncReadEntriesOrWait(int, AsyncCallbacks$ReadEntriesCallback, Object):594, ManagedCursorImpl (org.apache.bookkeeper.mledger.impl), ManagedCursorImpl.java
readMoreEntries():326, PersistentDispatcherMultipleConsumers (org.apache.pulsar.broker.service.persistent), PersistentDispatcherMultipleConsumers.java
consumerFlow(Consumer, int):240, PersistentDispatcherMultipleConsumers (org.apache.pulsar.broker.service.persistent), PersistentDispatcherMultipleConsumers.java
consumerFlow(Consumer, int):215, PersistentSubscription (org.apache.pulsar.broker.service.persistent), PersistentSubscription.java
flowPermits(int):376, Consumer (org.apache.pulsar.broker.service), Consumer.java
handleFlow(PulsarApi$CommandFlow):1090, ServerCnx (org.apache.pulsar.broker.service), ServerCnx.java
channelRead(ChannelHandlerContext, Object):160, PulsarDecoder (org.apache.pulsar.common.api), PulsarDecoder.java
invokeChannelRead(Object):362, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
invokeChannelRead(AbstractChannelHandlerContext, Object):348, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
fireChannelRead(Object):340, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
fireChannelRead(ChannelHandlerContext, CodecOutputList, int):323, ByteToMessageDecoder (io.netty.handler.codec), ByteToMessageDecoder.java
channelRead(ChannelHandlerContext, Object):297, ByteToMessageDecoder (io.netty.handler.codec), ByteToMessageDecoder.java
invokeChannelRead(Object):362, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
invokeChannelRead(AbstractChannelHandlerContext, Object):348, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
fireChannelRead(Object):340, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
channelRead(ChannelHandlerContext, Object):1434, DefaultChannelPipeline$HeadContext (io.netty.channel), DefaultChannelPipeline.java
invokeChannelRead(Object):362, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
invokeChannelRead(AbstractChannelHandlerContext, Object):348, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
fireChannelRead(Object):965, DefaultChannelPipeline (io.netty.channel), DefaultChannelPipeline.java
read():163, AbstractNioByteChannel$NioByteUnsafe (io.netty.channel.nio), AbstractNioByteChannel.java
processSelectedKey(SelectionKey, AbstractNioChannel):656, NioEventLoop (io.netty.channel.nio), NioEventLoop.java
processSelectedKeysOptimized():591, NioEventLoop (io.netty.channel.nio), NioEventLoop.java
processSelectedKeys():508, NioEventLoop (io.netty.channel.nio), NioEventLoop.java
run():470, NioEventLoop (io.netty.channel.nio), NioEventLoop.java
run():909, SingleThreadEventExecutor$5 (io.netty.util.concurrent), SingleThreadEventExecutor.java
run():30, FastThreadLocalRunnable (io.netty.util.concurrent), FastThreadLocalRunnable.java
run():748, Thread (java.lang), Thread.java
```

### org.apache.bookkeeper.mledger.impl.EntryCacheImpl
```
public boolean insert(EntryImpl entry) {
    if (!manager.hasSpaceInCache()) {
        if (log.isDebugEnabled()) {
            log.debug("[{}] Skipping cache while doing eviction: {} - size: {}", ml.getName(), entry.getPosition(),
                    entry.getLength());
        }
        return false;
    }

    if (log.isDebugEnabled()) {
        log.debug("[{}] Adding entry to cache: {} - size: {}", ml.getName(), entry.getPosition(),
                entry.getLength());
    }

    ByteBuf cachedData = null;
    if (copyEntries) {
        cachedData = copyEntry(entry);
        if (cachedData == null) {
            return false;
        }
    } else {
        // Use retain here to have the same counter increase as in the copy entry scenario
        cachedData = entry.getDataBuffer().retain();
    }

    PositionImpl position = entry.getPosition();
    EntryImpl cacheEntry = EntryImpl.create(position, cachedData);
    cachedData.release();
    if (entries.put(position, cacheEntry)) {
        manager.entryAdded(entry.getLength());
        return true;
    } else {
        // entry was not inserted into cache, we need to discard it
        cacheEntry.release();
        return false;
    }
}
```
