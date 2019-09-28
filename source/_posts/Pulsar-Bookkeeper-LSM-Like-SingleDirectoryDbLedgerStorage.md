---
title: Pulsar-Bookkeeper-LSM-Like-SingleDirectoryDbLedgerStorage
date: 2019-05-27 20:12:19
tags:
---


## 写入逻辑
## org.apache.bookkeeper.proto.WriteEntryProcessorV3#getAddResponse
```
private AddResponse getAddResponse() {
    final long startTimeNanos = MathUtils.nowInNano();
    AddRequest addRequest = request.getAddRequest();
    long ledgerId = addRequest.getLedgerId();
    long entryId = addRequest.getEntryId();

    final AddResponse.Builder addResponse = AddResponse.newBuilder()
            .setLedgerId(ledgerId)
            .setEntryId(entryId);

    if (!isVersionCompatible()) {
        addResponse.setStatus(StatusCode.EBADVERSION);
        return addResponse.build();
    }

    if (requestProcessor.getBookie().isReadOnly()
        && !(RequestUtils.isHighPriority(request)
                && requestProcessor.getBookie().isAvailableForHighPriorityWrites())) {
        logger.warn("BookieServer is running as readonly mode, so rejecting the request from the client!");
        addResponse.setStatus(StatusCode.EREADONLY);
        return addResponse.build();
    }

    // 这里构造了一个addEntry的callback，触发时机是，当Bookkeeper的journal成功将消息fsync到磁盘后（注意journalSyncData=true）
    BookkeeperInternalCallbacks.WriteCallback wcb = new BookkeeperInternalCallbacks.WriteCallback() {
        @Override
        public void writeComplete(int rc, long ledgerId, long entryId,
                                  BookieSocketAddress addr, Object ctx) {
            if (BookieProtocol.EOK == rc) {
                requestProcessor.getRequestStats().getAddEntryStats()
                    .registerSuccessfulEvent(MathUtils.elapsedNanos(startTimeNanos), TimeUnit.NANOSECONDS);
            } else {
                requestProcessor.getRequestStats().getAddEntryStats()
                    .registerFailedEvent(MathUtils.elapsedNanos(startTimeNanos), TimeUnit.NANOSECONDS);
            }

            StatusCode status;
            switch (rc) {
                case BookieProtocol.EOK:
                    status = StatusCode.EOK;
                    break;
                case BookieProtocol.EIO:
                    status = StatusCode.EIO;
                    break;
                default:
                    status = StatusCode.EUA;
                    break;
            }
            addResponse.setStatus(status);
            Response.Builder response = Response.newBuilder()
                    .setHeader(getHeader())
                    .setStatus(addResponse.getStatus())
                    .setAddResponse(addResponse);
            Response resp = response.build();
            sendResponse(status, resp, requestProcessor.getRequestStats().getAddRequestStats());
        }
    };
    final EnumSet<WriteFlag> writeFlags;
    if (addRequest.hasWriteFlags()) {
        writeFlags = WriteFlag.getWriteFlags(addRequest.getWriteFlags());
    } else {
        writeFlags = WriteFlag.NONE;
    }
    final boolean ackBeforeSync = writeFlags.contains(WriteFlag.DEFERRED_SYNC);
    StatusCode status = null;
    byte[] masterKey = addRequest.getMasterKey().toByteArray();
    ByteBuf entryToAdd = Unpooled.wrappedBuffer(addRequest.getBody().asReadOnlyByteBuffer());
    try {
        if (RequestUtils.hasFlag(addRequest, AddRequest.Flag.RECOVERY_ADD)) {
            requestProcessor.getBookie().recoveryAddEntry(entryToAdd, wcb, channel, masterKey);
        } else {
            requestProcessor.getBookie().addEntry(entryToAdd, ackBeforeSync, wcb, channel, masterKey);
        }
        status = StatusCode.EOK;
    } catch (OperationRejectedException e) {
        // Avoid to log each occurence of this exception as this can happen when the ledger storage is
        // unable to keep up with the write rate.
        if (logger.isDebugEnabled()) {
            logger.debug("Operation rejected while writing {}", request, e);
        }
        status = StatusCode.EIO;
    } catch (IOException e) {
        logger.error("Error writing entry:{} to ledger:{}",
                entryId, ledgerId, e);
        status = StatusCode.EIO;
    } catch (BookieException.LedgerFencedException e) {
        logger.error("Ledger fenced while writing entry:{} to ledger:{}",
                entryId, ledgerId, e);
        status = StatusCode.EFENCED;
    } catch (BookieException e) {
        logger.error("Unauthorized access to ledger:{} while writing entry:{}",
                ledgerId, entryId, e);
        status = StatusCode.EUA;
    } catch (Throwable t) {
        logger.error("Unexpected exception while writing {}@{} : ",
                entryId, ledgerId, t);
        // some bad request which cause unexpected exception
        status = StatusCode.EBADREQ;
    }

    // If everything is okay, we return null so that the calling function
    // doesn't return a response back to the caller.
    if (!status.equals(StatusCode.EOK)) {
        addResponse.setStatus(status);
        return addResponse.build();
    }
    return null;
}
```


## 读取逻辑
## org.apache.bookkeeper.bookie.storage.ldb.SingleDirectoryDbLedgerStorage#getEntry
```
public ByteBuf getEntry(long ledgerId, long entryId) throws IOException {
    long startTime = MathUtils.nowInNano();
    if (log.isDebugEnabled()) {
        log.debug("Get Entry: {}@{}", ledgerId, entryId);
    }

    if (entryId == BookieProtocol.LAST_ADD_CONFIRMED) {
        return getLastEntry(ledgerId);
    }

    // We need to try to read from both write caches, since recent entries could be found in either of the two. The
    // write caches are already thread safe on their own, here we just need to make sure we get references to both
    // of them. Using an optimistic lock since the read lock is always free, unless we're swapping the caches.
    long stamp = writeCacheRotationLock.tryOptimisticRead();
    WriteCache localWriteCache = writeCache;
    WriteCache localWriteCacheBeingFlushed = writeCacheBeingFlushed;
    if (!writeCacheRotationLock.validate(stamp)) {
        // Fallback to regular read lock approach
        stamp = writeCacheRotationLock.readLock();
        try {
            localWriteCache = writeCache;
            localWriteCacheBeingFlushed = writeCacheBeingFlushed;
        } finally {
            writeCacheRotationLock.unlockRead(stamp);
        }
    }

    // First try to read from the write cache of recent entries
    ByteBuf entry = localWriteCache.get(ledgerId, entryId);
    if (entry != null) {
        recordSuccessfulEvent(dbLedgerStorageStats.getReadCacheHitStats(), startTime);
        recordSuccessfulEvent(dbLedgerStorageStats.getReadEntryStats(), startTime);
        return entry;
    }

    // If there's a flush going on, the entry might be in the flush buffer
    entry = localWriteCacheBeingFlushed.get(ledgerId, entryId);
    if (entry != null) {
        recordSuccessfulEvent(dbLedgerStorageStats.getReadCacheHitStats(), startTime);
        recordSuccessfulEvent(dbLedgerStorageStats.getReadEntryStats(), startTime);
        return entry;
    }

    // Try reading from read-ahead cache
    entry = readCache.get(ledgerId, entryId);
    if (entry != null) {
        recordSuccessfulEvent(dbLedgerStorageStats.getReadCacheHitStats(), startTime);
        recordSuccessfulEvent(dbLedgerStorageStats.getReadEntryStats(), startTime);
        return entry;
    }

    // Read from main storage
    long entryLocation;
    try {
        entryLocation = entryLocationIndex.getLocation(ledgerId, entryId);
        if (entryLocation == 0) {
            throw new NoEntryException(ledgerId, entryId);
        }
        entry = entryLogger.readEntry(ledgerId, entryId, entryLocation);
    } catch (NoEntryException e) {
        recordFailedEvent(dbLedgerStorageStats.getReadEntryStats(), startTime);
        throw e;
    }

    readCache.put(ledgerId, entryId, entry);

    // Try to read more entries
    long nextEntryLocation = entryLocation + 4 /* size header */ + entry.readableBytes();
    fillReadAheadCache(ledgerId, entryId + 1, nextEntryLocation);

    recordSuccessfulEvent(dbLedgerStorageStats.getReadCacheMissStats(), startTime);
    recordSuccessfulEvent(dbLedgerStorageStats.getReadEntryStats(), startTime);
    return entry;
}
```