---
title: Kafka-FAQ
date: 2017-11-17 15:33:16
tags: Kafka
---

### Kafka写CommitLog时用了什么锁机制?

sync;lock-free;reentrant lock,用了哪一种？

### SkimpyOffsetMap 算是一个非常大的问题啊，万一消息被错误得compact怎么办呢？

### 假如我手动提交消息的offset，现在我业务处理失败，那么我不消费端不提交offset，那么Kafka会把这个消息怎么处理呀？后续消费端还是否能够消费到

### 日志truncate是怎么一回事？

1. ISR = {A, B, C}

A: Leader    B: Follower  C: Follower
|   m1  |    |   m1  |    |   m1  |
|   m2  |    |   m2  |
|   m3  |

2. A Crashes, New Leader: B

A: Crash     B: Leader    C: Follower
|   m1  |    |   m1  |    |   m1  |
|   m2  |    |   m2  |  > |   m2  |
|   m3  |

3. Send More messages

A: Crash     B: Leader    C: Follower
|   m1  |    |   m1  |    |   m1  |
|   m2  |    |   m2  |    |   m2  |
|   m3  |    |   m4  |  > |   m4  |
             |   m5  |  > |   m5  |
             |   m6  |  > |   m6  |

4. A back to work, 之前commit到m1,Truncate m1之后的所有数据(这一步很重要),这个逻辑需要关注
ISR = {B, C}

A: Folloer   B: Leader    C: Follower
|   m1  |    |   m1  |    |   m1  |
             |   m2  |    |   m2  |
             |   m4  |  > |   m4  |
             |   m5  |  > |   m5  |
             |   m6  |  > |   m6  |

5. A逐渐追上Leader
ISR = {A, B, C}

A: Folloer   B: Leader    C: Follower
|   m1  |    |   m1  |    |   m1  |
|   m2  |  < |   m2  |    |   m2  |
|   m4  |  < |   m4  |  > |   m4  |
|   m5  |  < |   m5  |  > |   m5  |
|   m6  |  < |   m6  |  > |   m6  |


第4步中，A恢复后，是主动要truncate日志，还是新的Leader要求它去truncate日志？

```
  @nonthreadsafe
  def truncateTo(offset: Long): Int = {
    // Do offset translation before truncating the index to avoid needless scanning
    // in case we truncate the full index
    val mapping = translateOffset(offset)
    offsetIndex.truncateTo(offset)
    timeIndex.truncateTo(offset)
    txnIndex.truncateTo(offset)

    // After truncation, reset and allocate more space for the (new currently active) index
    offsetIndex.resize(offsetIndex.maxIndexSize)
    timeIndex.resize(timeIndex.maxIndexSize)

    val bytesTruncated = if (mapping == null) 0 else log.truncateTo(mapping.position)
    if (log.sizeInBytes == 0) {
      created = time.milliseconds
      rollingBasedTimestamp = None
    }

    bytesSinceLastIndexEntry = 0
    if (maxTimestampSoFar >= 0)
      loadLargestTimestamp()
    bytesTruncated
  }
```



### 生产的消息何时被commit
producer：acks=-1， broker： min.isr=2
如果发送一条消息给leader，leader本地持久化，那需要等待至少一个除了自己之后的isr成功拉取到数据，并且给leader确认后，
leader才会返回给producer，这条消息已经成功发送了。
这个时候的high watermark为isr的最小LEO值，消费者最多只能看见这个值之前的消息。

### num.standby.replicas
如果本机的local state挂了，那么根据kafka的consume rebalance，会在另一台机器上恢复state，但是这个恢复时间
可能会比较长，所以有num.standby.replicas这个参数，这个到底是什么意思？

### kafka和RocketMQ设计导致得关键性差异——对多队列得支持程度

http://blog.csdn.net/chunlongyu/article/details/54576649


### 学习资料
http://blog.csdn.net/chunlongyu/article/category/6638499

http://blog.csdn.net/chunlongyu/article/details/54407633

http://blog.csdn.net/a417930422/article/category/6086259

http://blog.csdn.net/lizhitao

http://www.cnblogs.com/huxi2b

http://blog.csdn.net/u014393917/article/category/6332828

阿里中间件团队博客 
http://jm.taobao.org/categories/%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6/


第九课. Kafka高性能之道
    9.1 顺序写磁盘
    9.2 零拷贝
    9.3 批处理
    9.4 基于ISR的动态平衡一致性算法