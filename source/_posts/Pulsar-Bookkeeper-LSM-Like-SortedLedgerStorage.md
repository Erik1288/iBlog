---
title: Pulsar-Bookkeeper-LSM
date: 2019-05-20 15:44:41
tags:
---


## org.apache.bookkeeper.bookie.Bookie#addEntryInternal
```
/**
 * Add an entry to a ledger as specified by handle.
 */
private void addEntryInternal(LedgerDescriptor handle, ByteBuf entry,
                              boolean ackBeforeSync, WriteCallback cb, Object ctx, byte[] masterKey)
        throws IOException, BookieException, InterruptedException {
    long ledgerId = handle.getLedgerId();
    long entryId = handle.addEntry(entry);

    bookieStats.getWriteBytes().add(entry.readableBytes());

    // journal `addEntry` should happen after the entry is added to ledger storage.
    // otherwise the journal entry can potentially be rolled before the ledger is created in ledger storage.
    if (masterKeyCache.get(ledgerId) == null) {
        // Force the load into masterKey cache
        byte[] oldValue = masterKeyCache.putIfAbsent(ledgerId, masterKey);
        if (oldValue == null) {
            // new handle, we should add the key to journal ensure we can rebuild
            ByteBuffer bb = ByteBuffer.allocate(8 + 8 + 4 + masterKey.length);
            bb.putLong(ledgerId);
            bb.putLong(METAENTRY_ID_LEDGER_KEY);
            bb.putInt(masterKey.length);
            bb.put(masterKey);
            bb.flip();

            getJournal(ledgerId).logAddEntry(bb, false /* ackBeforeSync */, new NopWriteCallback(), null);
        }
    }

    if (LOG.isTraceEnabled()) {
        LOG.trace("Adding {}@{}", entryId, ledgerId);
    }
    getJournal(ledgerId).logAddEntry(entry, ackBeforeSync, cb, ctx);
}
```

## org.apache.bookkeeper.bookie.SortedLedgerStorage#addEntry
```
@Override
public long addEntry(ByteBuf entry) throws IOException {
    long ledgerId = entry.getLong(entry.readerIndex() + 0);
    long entryId = entry.getLong(entry.readerIndex() + 8);
    long lac = entry.getLong(entry.readerIndex() + 16);

    memTable.addEntry(ledgerId, entryId, entry.nioBuffer(), this);
    interleavedLedgerStorage.ledgerCache.updateLastAddConfirmed(ledgerId, lac);
    return entryId;
}
```

## org.apache.bookkeeper.bookie.EntryMemTable#addEntry
```
public long addEntry(long ledgerId, long entryId, final ByteBuffer entry, final CacheCallback cb)
        throws IOException {
    long size = 0;
    long startTimeNanos = MathUtils.nowInNano();
    boolean success = false;
    try {
        if (isSizeLimitReached() || (!previousFlushSucceeded.get())) {
            Checkpoint cp = snapshot();
            if ((null != cp) || (!previousFlushSucceeded.get())) {
                cb.onSizeLimitReached(cp);
            }
        }

        final int len = entry.remaining();
        if (!skipListSemaphore.tryAcquire(len)) {
            memTableStats.getThrottlingCounter().inc();
            final long throttlingStartTimeNanos = MathUtils.nowInNano();
            skipListSemaphore.acquireUninterruptibly(len);
            memTableStats.getThrottlingStats()
                .registerSuccessfulEvent(MathUtils.elapsedNanos(throttlingStartTimeNanos), TimeUnit.NANOSECONDS);
        }

        this.lock.readLock().lock();
        try {
            EntryKeyValue toAdd = cloneWithAllocator(ledgerId, entryId, entry);
            size = internalAdd(toAdd);
        } finally {
            this.lock.readLock().unlock();
        }
        success = true;
        return size;
    } finally {
        if (success) {
            memTableStats.getPutEntryStats()
                .registerSuccessfulEvent(MathUtils.elapsedNanos(startTimeNanos), TimeUnit.NANOSECONDS);
        } else {
            memTableStats.getPutEntryStats()
                .registerFailedEvent(MathUtils.elapsedNanos(startTimeNanos), TimeUnit.NANOSECONDS);
        }
    }
}
```

## org.apache.bookkeeper.bookie.EntryMemTable#snapshot(org.apache.bookkeeper.bookie.CheckpointSource.Checkpoint)
```
/**
 * Snapshot current EntryMemTable. if given <i>oldCp</i> is older than current checkpoint,
 * we don't do any snapshot. If snapshot happened, we return the checkpoint of the snapshot.
 *
 * @param oldCp
 *          checkpoint
 * @return checkpoint of the snapshot, null means no snapshot
 * @throws IOException
 */
Checkpoint snapshot(Checkpoint oldCp) throws IOException {
    Checkpoint cp = null;
    // No-op if snapshot currently has entries
    if (this.snapshot.isEmpty() && this.kvmap.compareTo(oldCp) < 0) {
        final long startTimeNanos = MathUtils.nowInNano();
        this.lock.writeLock().lock();
        try {
            if (this.snapshot.isEmpty() && !this.kvmap.isEmpty()
                    && this.kvmap.compareTo(oldCp) < 0) {
                this.snapshot = this.kvmap;
                this.kvmap = newSkipList();
                // get the checkpoint of the memtable.
                cp = this.kvmap.cp;
                // Reset heap to not include any keys
                this.size.set(0);
                // Reset allocator so we get a fresh buffer for the new EntryMemTable
                this.allocator = new SkipListArena(conf);
            }
        } finally {
            this.lock.writeLock().unlock();
        }

        if (null != cp) {
            memTableStats.getSnapshotStats()
                .registerSuccessfulEvent(MathUtils.elapsedNanos(startTimeNanos), TimeUnit.NANOSECONDS);
        } else {
            memTableStats.getSnapshotStats()
                .registerFailedEvent(MathUtils.elapsedNanos(startTimeNanos), TimeUnit.NANOSECONDS);
        }
    }
    return cp;
}
```

## org.apache.bookkeeper.bookie.SortedLedgerStorage#onSizeLimitReached
```
// CacheCallback functions.
@Override
public void onSizeLimitReached(final Checkpoint cp) throws IOException {
    LOG.info("Reached size {}", cp);
    // when size limit reached, we get the previous checkpoint from snapshot mem-table.
    // at this point, we are safer to schedule a checkpoint, since the entries added before
    // this checkpoint already written to entry logger.
    // but it would be better not to let mem-table flush to different entry log files,
    // so we roll entry log files in SortedLedgerStorage itself.
    // After that, we could make the process writing data to entry logger file not bound with checkpoint.
    // otherwise, it hurts add performance.
    //
    // The only exception for the size limitation is if a file grows to be more than hard limit 2GB,
    // we have to force rolling log, which it might cause slight performance effects
    scheduler.execute(new Runnable() {
        @Override
        public void run() {
            try {
                LOG.info("Started flushing mem table.");
                interleavedLedgerStorage.getEntryLogger().prepareEntryMemTableFlush();
                memTable.flush(SortedLedgerStorage.this);
                if (interleavedLedgerStorage.getEntryLogger().commitEntryMemTableFlush()) {
                    interleavedLedgerStorage.checkpointer.startCheckpoint(cp);
                }
            } catch (Exception e) {
                stateManager.transitionToReadOnlyMode();
                LOG.error("Exception thrown while flushing skip list cache.", e);
            }
        }
    });
}
```

## org.apache.bookkeeper.bookie.EntryMemTable#flush(org.apache.bookkeeper.bookie.SkipListFlusher)
```
/**
 * Flush snapshot and clear it.
 */
long flush(final SkipListFlusher flusher) throws IOException {
    try {
        long flushSize = flushSnapshot(flusher, Checkpoint.MAX);
        previousFlushSucceeded.set(true);
        return flushSize;
    } catch (IOException ioe) {
        previousFlushSucceeded.set(false);
        throw ioe;
    }
}
```

##