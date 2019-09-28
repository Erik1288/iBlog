---
title: Pulsar-Consume-Msg
date: 2019-06-13 19:11:45
tags:
---


### 居然是从FLOW开始的！！！

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


### 如果缓存中没有，那么就从bk中去读
```
private void asyncReadEntry0(ReadHandle lh, long firstEntry, long lastEntry, boolean isSlowestReader,
        final ReadEntriesCallback callback, Object ctx) {
    final long ledgerId = lh.getId();
    final int entriesToRead = (int) (lastEntry - firstEntry) + 1;
    final PositionImpl firstPosition = PositionImpl.get(lh.getId(), firstEntry);
    final PositionImpl lastPosition = PositionImpl.get(lh.getId(), lastEntry);

    if (log.isDebugEnabled()) {
        log.debug("[{}] Reading entries range ledger {}: {} to {}", ml.getName(), ledgerId, firstEntry, lastEntry);
    }

    Collection<EntryImpl> cachedEntries = entries.getRange(firstPosition, lastPosition);

    if (cachedEntries.size() == entriesToRead) {
        long totalCachedSize = 0;
        final List<EntryImpl> entriesToReturn = Lists.newArrayListWithExpectedSize(entriesToRead);

        // All entries found in cache
        for (EntryImpl entry : cachedEntries) {
            entriesToReturn.add(EntryImpl.create(entry));
            totalCachedSize += entry.getLength();
            entry.release();
        }

        manager.mlFactoryMBean.recordCacheHits(entriesToReturn.size(), totalCachedSize);
        if (log.isDebugEnabled()) {
            log.debug("[{}] Ledger {} -- Found in cache entries: {}-{}", ml.getName(), ledgerId, firstEntry,
                    lastEntry);
        }

        callback.readEntriesComplete((List) entriesToReturn, ctx);

    } else {
        if (!cachedEntries.isEmpty()) {
            cachedEntries.forEach(entry -> entry.release());
        }

        // Read all the entries from bookkeeper
        lh.readAsync(firstEntry, lastEntry).whenCompleteAsync(
                (ledgerEntries, exception) -> {
                    if (exception != null) {
                        if (exception instanceof BKException
                            && ((BKException)exception).getCode() == BKException.Code.TooManyRequestsException) {
                            callback.readEntriesFailed(createManagedLedgerException(exception), ctx);
                        } else {
                            ml.invalidateLedgerHandle(lh, exception);
                            ManagedLedgerException mlException = createManagedLedgerException(exception);
                            callback.readEntriesFailed(mlException, ctx);
                        }
                        return;
                    }

                    checkNotNull(ml.getName());
                    checkNotNull(ml.getExecutor());

                    try {
                        // We got the entries, we need to transform them to a List<> type
                        long totalSize = 0;
                        final List<EntryImpl> entriesToReturn
                            = Lists.newArrayListWithExpectedSize(entriesToRead);
                        for (LedgerEntry e : ledgerEntries) {
                            EntryImpl entry = EntryImpl.create(e);

                            entriesToReturn.add(entry);
                            totalSize += entry.getLength();
                        }

                        manager.mlFactoryMBean.recordCacheMiss(entriesToReturn.size(), totalSize);
                        ml.getMBean().addReadEntriesSample(entriesToReturn.size(), totalSize);

                        callback.readEntriesComplete((List) entriesToReturn, ctx);
                    } finally {
                        ledgerEntries.close();
                    }
                }, ml.getExecutor().chooseThread(ml.getName()));
    }
}
```