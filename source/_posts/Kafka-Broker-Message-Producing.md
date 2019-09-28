---
title: Kafka-Broker-Message-Producing
date: 2019-08-05 11:48:49
tags: Kafka
---

## 这个过程需要验证这个批次里面的每一个消息的CRC和size，也就是说要解压么？
```
private def analyzeAndValidateRecords(records: MemoryRecords, isFromClient: Boolean): LogAppendInfo = {
    var shallowMessageCount = 0
    var validBytesCount = 0
    var firstOffset: Option[Long] = None
    var lastOffset = -1L
    var sourceCodec: CompressionCodec = NoCompressionCodec
    var monotonic = true
    var maxTimestamp = RecordBatch.NO_TIMESTAMP
    var offsetOfMaxTimestamp = -1L
    var readFirstMessage = false
    var lastOffsetOfFirstBatch = -1L

    for (batch <- records.batches.asScala) {
      // we only validate V2 and higher to avoid potential compatibility issues with older clients
      if (batch.magic >= RecordBatch.MAGIC_VALUE_V2 && isFromClient && batch.baseOffset != 0)
        throw new InvalidRecordException(s"The baseOffset of the record batch in the append to $topicPartition should " +
          s"be 0, but it is ${batch.baseOffset}")

      // update the first offset if on the first message. For magic versions older than 2, we use the last offset
      // to avoid the need to decompress the data (the last offset can be obtained directly from the wrapper message).
      // For magic version 2, we can get the first offset directly from the batch header.
      // When appending to the leader, we will update LogAppendInfo.baseOffset with the correct value. In the follower
      // case, validation will be more lenient.
      // Also indicate whether we have the accurate first offset or not
      if (!readFirstMessage) {
        if (batch.magic >= RecordBatch.MAGIC_VALUE_V2)
          firstOffset = Some(batch.baseOffset)
        lastOffsetOfFirstBatch = batch.lastOffset
        readFirstMessage = true
      }

      // check that offsets are monotonically increasing
      if (lastOffset >= batch.lastOffset)
        monotonic = false

      // update the last offset seen
      lastOffset = batch.lastOffset

      // Check if the message sizes are valid.
      val batchSize = batch.sizeInBytes
      if (batchSize > config.maxMessageSize) {
        brokerTopicStats.topicStats(topicPartition.topic).bytesRejectedRate.mark(records.sizeInBytes)
        brokerTopicStats.allTopicsStats.bytesRejectedRate.mark(records.sizeInBytes)
        throw new RecordTooLargeException(s"The record batch size in the append to $topicPartition is $batchSize bytes " +
          s"which exceeds the maximum configured value of ${config.maxMessageSize}.")
      }

      // check the validity of the message by checking CRC
      batch.ensureValid()

      if (batch.maxTimestamp > maxTimestamp) {
        maxTimestamp = batch.maxTimestamp
        offsetOfMaxTimestamp = lastOffset
      }

      shallowMessageCount += 1
      validBytesCount += batchSize

      val messageCodec = CompressionCodec.getCompressionCodec(batch.compressionType.id)
      if (messageCodec != NoCompressionCodec)
        sourceCodec = messageCodec
    }

    // Apply broker-side compression if any
    val targetCodec = BrokerCompressionCodec.getTargetCompressionCodec(config.compressionType, sourceCodec)
    LogAppendInfo(firstOffset, lastOffset, maxTimestamp, offsetOfMaxTimestamp, RecordBatch.NO_TIMESTAMP, logStartOffset,
      RecordConversionStats.EMPTY, sourceCodec, targetCodec, shallowMessageCount, validBytesCount, monotonic, lastOffsetOfFirstBatch)
  }
```


<init>(ByteBuffer):57, MemoryRecords (org.apache.kafka.common.record), MemoryRecords.java
readableRecords(ByteBuffer):425, MemoryRecords (org.apache.kafka.common.record), MemoryRecords.java
read(ByteBuffer):561, Type$11 (org.apache.kafka.common.protocol.types), Type.java
read(ByteBuffer):544, Type$11 (org.apache.kafka.common.protocol.types), Type.java
read(ByteBuffer):106, Schema (org.apache.kafka.common.protocol.types), Schema.java
read(ByteBuffer):27, Schema (org.apache.kafka.common.protocol.types), Schema.java
read(ByteBuffer):78, ArrayOf (org.apache.kafka.common.protocol.types), ArrayOf.java
read(ByteBuffer):106, Schema (org.apache.kafka.common.protocol.types), Schema.java
read(ByteBuffer):27, Schema (org.apache.kafka.common.protocol.types), Schema.java
read(ByteBuffer):78, ArrayOf (org.apache.kafka.common.protocol.types), ArrayOf.java
read(ByteBuffer):106, Schema (org.apache.kafka.common.protocol.types), Schema.java
parseRequest(short, ByteBuffer):297, ApiKeys (org.apache.kafka.common.protocol), ApiKeys.java
parseRequest(ByteBuffer):63, RequestContext (org.apache.kafka.common.requests), RequestContext.java
<init>(int, RequestContext, long, MemoryPool, ByteBuffer, RequestChannel$Metrics):89, RequestChannel$Request (kafka.network), RequestChannel.scala
$anonfun$processCompletedReceives$1(Processor, NetworkReceive):891, Processor (kafka.network), SocketServer.scala
$anonfun$processCompletedReceives$1$adapted(Processor, NetworkReceive):873, Processor (kafka.network), SocketServer.scala
apply(Object):-1, 214985191 (kafka.network.Processor$$Lambda$686), Unknown Source
foreach(Function1):941, Iterator (scala.collection), Iterator.scala
foreach$(Iterator, Function1):941, Iterator (scala.collection), Iterator.scala
foreach(Function1):1429, AbstractIterator (scala.collection), Iterator.scala
foreach(Function1):74, IterableLike (scala.collection), IterableLike.scala
foreach$(IterableLike, Function1):73, IterableLike (scala.collection), IterableLike.scala
foreach(Function1):56, AbstractIterable (scala.collection), Iterable.scala
processCompletedReceives():873, Processor (kafka.network), SocketServer.scala
run():763, Processor (kafka.network), SocketServer.scala
run():748, Thread (java.lang), Thread.java