---
title: Kafka-FAQ
date: 2017-11-17 15:33:16
tags: Kafka
---
### 大量精品文章和面试问题
https://blog.csdn.net/u013256816
比较好的总结
https://juejin.im/post/5ddf5659518825782d599641?utm_source=gold_browser_extension

### Kafka发送怎么保证顺序？

### Kafka异步发送能不能保证顺序？

### Kafka能不能保证不丢消息（只要多数机器不挂）？

### Kafka能不能保证消费顺序？

### Consumer消费2个Partition，两个partition都有大量的数量，如果Poll的时候取500条，那么取到的是第一个partition的消息，还是第二个，还有两个都有？某一个会不会被饿死？

### Kafka写CommitLog时用了什么锁机制?

sync;lock-free;reentrant lock,用了哪一种？

kafka.log.Log#append
lock synchronized {
}

### Kafka生产者批量发送了消息，那Broker是把消息一条一存么？
不是，是把几条连续的消息存在一起，在外层公用同一个offset

### kafka消费者fetch的消息正好在某个批次中间的消息，怎么办？


### Kafka Consumer Rebalance流程是怎么样的？
1. Consumer查找GroupCoordinator，向它发送Join请求
2. Broker收到Join请求后，创建一个Group，并且把创建的Consumer作为Consumer的Leader
```
// @ kafka.coordinator.group.GroupMetadata#add
def add(member: MemberMetadata) {
  if (members.isEmpty)
    this.protocolType = Some(member.protocolType)

  assert(groupId == member.groupId)
  assert(this.protocolType.orNull == member.protocolType)
  assert(supportsProtocols(member.protocols))

  if (leaderId.isEmpty)
    leaderId = Some(member.memberId)
  members.put(member.memberId, member)
}
```
3. Broker并不会直接给发送Join请求的Consumer响应，而是会启动延时任务，等待一段时间，然后再给所有的Consumer响应
```
// @ kafka.coordinator.group.GroupCoordinator#prepareRebalance
private def prepareRebalance(group: GroupMetadata) {
  // if any members are awaiting sync, cancel their request and have them rejoin
  if (group.is(AwaitingSync))
    resetAndPropagateAssignmentError(group, Errors.REBALANCE_IN_PROGRESS)

  val delayedRebalance = if (group.is(Empty))
    new InitialDelayedJoin(this,
      joinPurgatory,
      group,
      groupConfig.groupInitialRebalanceDelayMs,
      groupConfig.groupInitialRebalanceDelayMs,
      max(group.rebalanceTimeoutMs - groupConfig.groupInitialRebalanceDelayMs, 0))
  else
    new DelayedJoin(this, group, group.rebalanceTimeoutMs)

  group.transitionTo(PreparingRebalance)

  info(s"Preparing to rebalance group ${group.groupId} with old generation ${group.generationId} " +
    s"(${Topic.GROUP_METADATA_TOPIC_NAME}-${partitionFor(group.groupId)})")

  val groupKey = GroupKey(group.groupId)
  joinPurgatory.tryCompleteElseWatch(delayedRebalance, Seq(groupKey))
}
```
4. 各个Consumer收到来自Group coordinator的响应后，会查看自己是不是这个Consumer Group中的Consumer Leader
```
// @ org.apache.kafka.clients.consumer.internals.AbstractCoordinator.JoinGroupResponseHandler#handle
AbstractCoordinator.this.generation = new Generation(joinResponse.generationId(),
        joinResponse.memberId(), joinResponse.groupProtocol());
if (joinResponse.isLeader()) {
    onJoinLeader(joinResponse).chain(future);
} else {
    onJoinFollower().chain(future);
}
```
5. 作为Leader的Consumer需要根据自己Rebalance算法，把rebalance的结果通过发送Sync请求反馈给Group Coordinator
```
// @ org.apache.kafka.clients.consumer.internals.AbstractCoordinator#onJoinLeader
private RequestFuture<ByteBuffer> onJoinLeader(JoinGroupResponse joinResponse) {
    try {
        // perform the leader synchronization and send back the assignment for the group
        Map<String, ByteBuffer> groupAssignment = performAssignment(joinResponse.leaderId(), joinResponse.groupProtocol(),
                joinResponse.members());

        SyncGroupRequest.Builder requestBuilder =
                new SyncGroupRequest.Builder(groupId, generation.generationId, generation.memberId, groupAssignment);
        log.debug("Sending leader SyncGroup to coordinator {}: {}", this.coordinator, requestBuilder);
        return sendSyncGroupRequest(requestBuilder);
    } catch (RuntimeException e) {
        return RequestFuture.failure(e);
    }
}
```
6. 而那些普通的Consumer也需要发送一个空的Sync请求
```
// @ org.apache.kafka.clients.consumer.internals.AbstractCoordinator#onJoinFollower
private RequestFuture<ByteBuffer> onJoinFollower() {
    // send follower's sync group with an empty assignment
    SyncGroupRequest.Builder requestBuilder =
            new SyncGroupRequest.Builder(groupId, generation.generationId, generation.memberId,
                    Collections.<String, ByteBuffer>emptyMap());
    log.debug("Sending follower SyncGroup to coordinator {}: {}", this.coordinator, requestBuilder);
    return sendSyncGroupRequest(requestBuilder);
}
```
7. 所有的Consumer根据Sync请求的响应，更新自己的
```
// @ org.apache.kafka.clients.consumer.internals.AbstractCoordinator.SyncGroupResponseHandler#handle
Errors error = syncResponse.error();
if (error == Errors.NONE) {
    sensors.syncLatency.record(response.requestLatencyMs());
    future.complete(syncResponse.memberAssignment());
}

// @ org.apache.kafka.clients.consumer.internals.AbstractCoordinator#joinGroupIfNeeded
void joinGroupIfNeeded() {
    while (needRejoin() || rejoinIncomplete()) {
        // ...

        RequestFuture<ByteBuffer> future = initiateJoinGroup();
        client.poll(future);

        if (future.succeeded()) {
            onJoinComplete(generation.generationId, generation.memberId, generation.protocol, future.value());

            // We reset the join group future only after the completion callback returns. This ensures
            // that if the callback is woken up, we will retry it on the next joinGroupIfNeeded.
            resetJoinGroupFuture();
            needsJoinPrepare = true;
        } else {
            // ...
        }
    }
}

// @ org.apache.kafka.clients.consumer.internals.ConsumerCoordinator#onJoinComplete
protected void onJoinComplete(int generation,
                              String memberId,
                              String assignmentStrategy,
                              ByteBuffer assignmentBuffer) {
    // ...
    Assignment assignment = ConsumerProtocol.deserializeAssignment(assignmentBuffer);

    // ...

    // update partition assignment
    subscriptions.assignFromSubscribed(assignment.partitions());

    // ...
}
```

### 怎么监控kafka page cache刷盘时间？


### Kafka shallowOffset是什么意思？

### Kafka的Retention是怎么工作的？不符合retention的日志什么时候会被清理掉？

### Kafka发送者怎么保证是有序的？
MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION = 1， 为什么？
不等于1的话，会有什么效果？

### Consumer coordinator是什么？

### Group coordinator是什么？

### __consumer_offset的某一个partition挂了，kafka broker中的controller会给它选一个新的Leader，这个过程是怎么样的？

### 怎么获取Kafka Consumer的Lag

### Kafka是一个batch一压缩，还是一条消息一压缩？
每一个消息都会压缩

### Kafka是怎么实现幂等的？
Broker以(producer, topic, partition)为维度，维护一份Map</*(producer, topic, partition)*/, /*sequence*/>，Producer每发送一条消息(好像是以Batch为单位的??)，都会将sequence++；如果同一个Producer对于某一个(topic, partition)发送了两个sequence一样的消息，后面发送的那个将被丢弃掉。
实现原理：
)Producer在初始化时，会向Broker申请一个ProducerId
)Broker处理ProducerId申请请求，从序列段中申请一个唯一ID
)Producer在发送消息时，会将加一的sequence发在消息体中，一起发送
```
// @ org.apache.kafka.clients.producer.internals.RecordAccumulator#drain

// Additionally, we update the next sequence number bound for the partition,
// and also have the transaction manager track the batch so as to ensure
// that sequence ordering is maintained even if we receive out of order
// responses.
batch.setProducerState(producerIdAndEpoch, transactionManager.sequenceNumber(batch.topicPartition), isTransactional);


// @ org.apache.kafka.common.record.DefaultRecordBatch#incrementSequence
static int incrementSequence(int baseSequence, int increment) {
    if (baseSequence > Integer.MAX_VALUE - increment)
        return increment - (Integer.MAX_VALUE - baseSequence) - 1;
    return baseSequence + increment;
}
```
)Broker如果收到了sequence一样的消息，丢弃后直接返回。
```
// @ 
// if this is a client produce request, there will be up to 5 batches which could have been duplicated.
// If we find a duplicate, we return the metadata of the appended batch to the client.
if (isFromClient) {
  maybeLastEntry.flatMap(_.findDuplicateBatch(batch)).foreach { duplicate =>
    return (updatedProducers, completedTxns.toList, Some(duplicate))
  }
}
```


### Kafka Consumer关闭后，怎么触发Rebalance？
``` 
// @ org.apache.kafka.clients.consumer.internals.ConsumerCoordinator#needRejoin
public boolean needRejoin() {
    if (!subscriptions.partitionsAutoAssigned())
        return false;

    // we need to rejoin if we performed the assignment and metadata has changed
    if (assignmentSnapshot != null && !assignmentSnapshot.equals(metadataSnapshot))
        return true;

    // we need to join if our subscription has changed since the last join
    if (joinedSubscription != null && !joinedSubscription.equals(subscriptions.subscription()))
        return true;

    return super.needRejoin();
}
```

### Kafka创建Topic的过程，和我们想象中有点不一样
1. 生成(topic, partition)replica的assignment
```
def createTopic(zkUtils: ZkUtils,
                topic: String,
                partitions: Int,
                replicationFactor: Int,
                topicConfig: Properties = new Properties,
                rackAwareMode: RackAwareMode = RackAwareMode.Enforced) {
  val brokerMetadatas = getBrokerMetadatas(zkUtils, rackAwareMode)
  val replicaAssignment = AdminUtils.assignReplicasToBrokers(brokerMetadatas, partitions, replicationFactor)
  AdminUtils.createOrUpdateTopicPartitionAssignmentPathInZK(zkUtils, topic, replicaAssignment, topicConfig)
}
```
2. 将生成的replica assignment写到ZK上
```
// create the partition assignment
writeTopicPartitionAssignment(zkUtils, topic, partitionReplicaAssignment, update)
```
3. Broker监听到这个ZK变化，然后开始调用状态机，使replica和topic上线
```
case class TopicChange(topics: Set[String]) extends ControllerEvent {

  def state = ControllerState.TopicChange

  override def process(): Unit = {
    if (!isActive) return
    val newTopics = topics -- controllerContext.allTopics
    val deletedTopics = controllerContext.allTopics -- topics
    controllerContext.allTopics = topics

    val addedPartitionReplicaAssignment = zkUtils.getReplicaAssignmentForTopics(newTopics.toSeq)
    controllerContext.partitionReplicaAssignment = controllerContext.partitionReplicaAssignment.filter(p =>
      !deletedTopics.contains(p._1.topic))
    controllerContext.partitionReplicaAssignment.++=(addedPartitionReplicaAssignment)
    info("New topics: [%s], deleted topics: [%s], new partition replica assignment [%s]".format(newTopics,
      deletedTopics, addedPartitionReplicaAssignment))
    if (newTopics.nonEmpty)
      onNewTopicCreation(newTopics, addedPartitionReplicaAssignment.keySet)
  }
}

/**
  * This callback is invoked by the topic change callback with the list of failed brokers as input.
  * It does the following -
  * 1. Move the newly created partitions to the NewPartition state
  * 2. Move the newly created partitions from NewPartition->OnlinePartition state
  */
def onNewPartitionCreation(newPartitions: Set[TopicAndPartition]) {
  info("New partition creation callback for %s".format(newPartitions.mkString(",")))
  partitionStateMachine.handleStateChanges(newPartitions, NewPartition)
  replicaStateMachine.handleStateChanges(controllerContext.replicasForPartition(newPartitions), NewReplica)
  partitionStateMachine.handleStateChanges(newPartitions, OnlinePartition, offlinePartitionSelector)
  replicaStateMachine.handleStateChanges(controllerContext.replicasForPartition(newPartitions), OnlineReplica)
}
```

### Kafka的__consumer_offset一共有50个partition，那么我指定的某一个(topic, partition)的是被这50个__consumer_offset中的哪个管理的？
```
case FindCoordinatorRequest.CoordinatorType.GROUP =>
  val partition = groupCoordinator.partitionFor(findCoordinatorRequest.coordinatorKey)
  val metadata = getOrCreateInternalTopic(GROUP_METADATA_TOPIC_NAME, request.context.listenerName)
  (partition, metadata)
```
http://matt33.com/2017/10/22/consumer-join-group/

### Kafka中有哪些功能是这样的模式：1. 修改ZK； 2. Broker监听到ZK变化； 3. Broker 根据ZK变化进行逻辑响应
1. Topic创建时，replica的assignment
2. Topic的Reassignment


### Kafka Consumer消费流程
1. rebalance
2. 获取assign的(topic, partition)的消费位点
3. 

### Kafka Broker在做"Join Request"处理时，需要等待一个timeout的时间，好让所有相关的consumer都能来得及join进来？这个是怎么实现的？
```
private def prepareRebalance(group: GroupMetadata) {
  // if any members are awaiting sync, cancel their request and have them rejoin
  if (group.is(AwaitingSync))
    resetAndPropagateAssignmentError(group, Errors.REBALANCE_IN_PROGRESS)

  val delayedRebalance = if (group.is(Empty))
    new InitialDelayedJoin(this,
      joinPurgatory,
      group,
      groupConfig.groupInitialRebalanceDelayMs,
      groupConfig.groupInitialRebalanceDelayMs,
      max(group.rebalanceTimeoutMs - groupConfig.groupInitialRebalanceDelayMs, 0))
  else
    new DelayedJoin(this, group, group.rebalanceTimeoutMs)

  group.transitionTo(PreparingRebalance)

  info(s"Preparing to rebalance group ${group.groupId} with old generation ${group.generationId} " +
    s"(${Topic.GROUP_METADATA_TOPIC_NAME}-${partitionFor(group.groupId)})")

  val groupKey = GroupKey(group.groupId)
  joinPurgatory.tryCompleteElseWatch(delayedRebalance, Seq(groupKey))
}
```

### 如果(topic, partiton)的leader在Broker3，Producer是怎么发送消息给Broker3的，是需要在启动的时候连接所有的Broker么？

### 如果需要消费在Broker2上的（topic, partition），Consumer是怎么去连接Broker2的，Consumer会连接整个集群所有的Broker么？
1. 先找GroupCoordinator，连接一台负载最低的机器，发送`Find Coordinator`命令
```
protected synchronized RequestFuture<Void> lookupCoordinator() {
    if (findCoordinatorFuture == null) {
        // find a node to ask about the coordinator
        Node node = this.client.leastLoadedNode();
        if (node == null) {
            log.debug("No broker available to send FindCoordinator request");
            return RequestFuture.noBrokersAvailable();
        } else
            findCoordinatorFuture = sendFindCoordinatorRequest(node);
    }
    return findCoordinatorFuture;
}
```

### SkimpyOffsetMap 算是一个非常大的问题啊，万一消息被错误得compact怎么办呢？

### 假如我手动提交消息的offset，现在我业务处理失败，那么我消费端不提交offset，那么Kafka会把这个消息怎么处理呀？后续消费端还是否能够消费到？


### 当某一个Partition的Leader挂了，Controller会为其选出一个新的Leader，这时是ISR中随便选一个么？
按照我的理解，应该是去选一个LEO最大的吧，完全错误
请分析这段代码
参考这篇：
https://www.jianshu.com/p/13548893bf31
http://ifeve.com/kafka-controller/

真实日志
```
kafka1/controller.log:[2019-10-04 08:26:01,376] DEBUG [PartitionStateMachine controllerId=1] After leader election, leader cache for krp-0 is updated to (Leader:2,ISR:2,1,LeaderEpoch:1,ControllerEpoch:1) (kafka.controller.PartitionStateMachine)
➜  /tmp grep -R ": current leader =" kafka*
kafka1/controller.log:[2019-10-04 08:26:01,313] DEBUG [ControlledShutdownLeaderSelector]: Partition krp-0 : current leader = 3, new leader = 2 (kafka.controller.ControlledShutdownLeaderSelector)


in kafka3/controller.log
[2019-10-04 08:26:02,132] INFO [controller-event-thread]: Shutting down (kafka.controller.ControllerEventManager$ControllerEventThread)
[2019-10-04 08:26:02,132] INFO [controller-event-thread]: Shutdown completed (kafka.controller.ControllerEventManager$ControllerEventThread)
[2019-10-04 08:26:02,133] INFO [controller-event-thread]: Stopped (kafka.controller.ControllerEventManager$ControllerEventThread)
[2019-10-04 08:26:02,133] DEBUG [Controller id=3] Resigning (kafka.controller.KafkaController)
[2019-10-04 08:26:02,133] DEBUG [Controller id=3] De-registering IsrChangeNotificationListener (kafka.controller.KafkaController)
[2019-10-04 08:26:02,134] DEBUG [Controller id=3] De-registering logDirEventNotificationListener (kafka.controller.KafkaController)
[2019-10-04 08:26:02,135] INFO [PartitionStateMachine controllerId=3] Stopped partition state machine (kafka.controller.PartitionStateMachine)
[2019-10-04 08:26:02,136] INFO [ReplicaStateMachine controllerId=3] Stopped replica state machine (kafka.controller.ReplicaStateMachine)
[2019-10-04 08:26:02,137] INFO [Controller id=3] Resigned (kafka.controller.KafkaController)

in kafka1/controller.log
[2019-10-04 08:26:01,300] INFO [Controller id=1] Shutting down broker 3 (kafka.controller.KafkaController)
[2019-10-04 08:26:01,301] DEBUG [Controller id=1] All shutting down brokers: 3 (kafka.controller.KafkaController)
[2019-10-04 08:26:01,302] DEBUG [Controller id=1] Live brokers: 1,2 (kafka.controller.KafkaController)
[2019-10-04 08:26:01,308] INFO [PartitionStateMachine controllerId=1] Invoking state change to OnlinePartition for partitions krp-0 (kafka.controller.PartitionStateMachine)
[2019-10-04 08:26:01,313] DEBUG [ControlledShutdownLeaderSelector]: Partition krp-0 : current leader = 3, new leader = 2 (kafka.controller.ControlledShutdownLeaderSelector)
[2019-10-04 08:26:01,376] DEBUG [PartitionStateMachine controllerId=1] After leader election, leader cache for krp-0 is updated to (Leader:2,ISR:2,1,LeaderEpoch:1,ControllerEpoch:1) (kafka.controller.PartitionStateMachine)

in kafka1/state-change.log
kafka1/state-change.log:[2019-10-04 08:26:01,376] TRACE [Controller id=1 epoch=1] Changed partition krp-0 from OnlinePartition to OnlinePartition with leader 2 (state.change.logger)
```
class ControlledShutdownLeaderSelector(controllerContext: ControllerContext) extends PartitionLeaderSelector with Logging {

  logIdent = "[ControlledShutdownLeaderSelector]: "

  def selectLeader(topicAndPartition: TopicAndPartition,
                   currentLeaderAndIsr: LeaderAndIsr): (LeaderAndIsr, Seq[Int]) = {
    val currentIsr = currentLeaderAndIsr.isr
    val assignedReplicas = controllerContext.partitionReplicaAssignment(topicAndPartition)
    val liveAssignedReplicas = assignedReplicas.filter(r => controllerContext.isReplicaOnline(r, topicAndPartition, true))

    val newIsr = currentIsr.filter(brokerId => !controllerContext.shuttingDownBrokerIds.contains(brokerId))
    liveAssignedReplicas.find(newIsr.contains) match {
      case Some(newLeader) =>
        debug(s"Partition $topicAndPartition : current leader = ${currentLeaderAndIsr.leader}, new leader = $newLeader")
        val newLeaderAndIsr = currentLeaderAndIsr.newLeaderAndIsr(newLeader, newIsr)
        (newLeaderAndIsr, liveAssignedReplicas)
      case None =>
        throw new StateChangeFailedException(s"No other replicas in ISR ${currentIsr.mkString(",")} for $topicAndPartition " +
          s"besides shutting down brokers ${controllerContext.shuttingDownBrokerIds.mkString(",")}")
    }
  }
}


### Consumer怎么去监控Rebalance事件？
添加ConsumerRebalanceListener监听器

### 为什么要引入epoch？
这个KIP写得非常清楚。
原先Truncate到HighWatermark处，但是这样做是有问题的，原因是Follower的HighWatermark是在消息存储后的下一轮RPC才获取到Leader的HighWatermark（Follower把消息接收后，然后发起FetchRPC告知Leader，Leader将自己的HighWatermark+1，这时Leader的返回值中含有最新的HighWatermark，但是如果这时Leader没有新的数据，这个Fetch请求就是一个长轮询请求，Follower将会延迟一段时间拿到最新的HighWatermark），这时如果Follower重启了，它将会把Leader已经确认过的HighWatermark对应的数据给Truncate掉。如果Leader这时也挂了，那么Follower将会成为新的Leader，中间的数据将会对消费者永久丢失。

https://cwiki.apache.org/confluence/display/KAFKA/KIP-101+-+Alter+Replication+Protocol+to+use+Leader+Epoch+rather+than+High+Watermark+for+Truncation#KIP-101-AlterReplicationProtocoltouseLeaderEpochratherthanHighWatermarkforTruncation-Scenario1:HighWatermarkTruncationfollowedbyImmediateLeaderElection

### 日志truncate是怎么一回事？

1. ISR = {A, B, C}, HighWatermark = m1

A: Leader    B: Follower  C: Follower
|   m1  |    |   m1  |    |   m1  |
|   m2  |    |   m2  |
|   m3  |

2. A Crashes, New Leader: B, Question:这里C有可能成为Leader么？

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


第4步中，A恢复后，是主动要truncate日志，还是新的Leader要求它去truncate日志？看下面的实现，貌似是follower会主动根据leader的epoch进行日志切割

```
/**
    * - Truncate the log to the leader's offset for each partition's epoch.
    * - If the leader's offset is greater, we stick with the Log End Offset
    *   otherwise we truncate to the leaders offset.
    * - If the leader replied with undefined epoch offset we must use the high watermark
    */
  override def maybeTruncate(fetchedEpochs: Map[TopicPartition, EpochEndOffset]): ResultWithPartitions[Map[TopicPartition, Long]] = {
    val truncationPoints = scala.collection.mutable.HashMap.empty[TopicPartition, Long]
    val partitionsWithError = mutable.Set[TopicPartition]()

    fetchedEpochs.foreach { case (tp, epochOffset) =>
      try {
        val replica = replicaMgr.getReplicaOrException(tp)

        if (epochOffset.hasError) {
          info(s"Retrying leaderEpoch request for partition ${replica.topicPartition} as the leader reported an error: ${epochOffset.error}")
          partitionsWithError += tp
        } else {
          val truncationOffset =
            if (epochOffset.endOffset == UNDEFINED_EPOCH_OFFSET)
              highWatermark(replica, epochOffset)
            else if (epochOffset.endOffset >= replica.logEndOffset.messageOffset)
              logEndOffset(replica, epochOffset)
            else
              epochOffset.endOffset

          replicaMgr.logManager.truncateTo(Map(tp -> truncationOffset))
          truncationPoints.put(tp, truncationOffset)
        }
      } catch {
        case e: KafkaStorageException =>
          info(s"Failed to truncate $tp", e)
          partitionsWithError += tp
      }
    }

    ResultWithPartitions(truncationPoints, partitionsWithError)
  }
```

```
查找当前leader_epoch的最后一个offset
override def endOffsetFor(requestedEpoch: Int): Long = {
    inReadLock(lock) {
      val offset =
        if (requestedEpoch == UNDEFINED_EPOCH) {
          // this may happen if a bootstrapping follower sends a request with undefined epoch or
          // a follower is on the older message format where leader epochs are not recorded
          UNDEFINED_EPOCH_OFFSET
        } else if (requestedEpoch == latestEpoch) {
          leo().messageOffset
        } else {
          val subsequentEpochs = epochs.filter(e => e.epoch > requestedEpoch)
          if (subsequentEpochs.isEmpty || requestedEpoch < epochs.head.epoch)
            UNDEFINED_EPOCH_OFFSET
          else
            subsequentEpochs.head.startOffset
        }
      debug(s"Processed offset for epoch request for partition ${topicPartition} epoch:$requestedEpoch and returning offset $offset from epoch list of size ${epochs.size}")
      offset
    }
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

### 这个过程需要验证这个批次里面的每一个消息的CRC和size，也就是说要解压么？

### 在没有Zookeeper的情况下使用Kafka吗？

### 某一个(Topic, Partition)设置了3个Replica，分别在broker1, broker2, broker3上，对于每一个broker，它们的AR（assign replica）是一样的么？
broker1: Leader(1), AR(1, 2, 3)
broker2: Follower(2), AR(1, 2, 3)
broker3: Follower(3), AR(1, 2, 3)

### Follower向Leader发送Fetch请求，HW是怎么在Follower和Leader之间传递的？

### 什么叫某个(topic, partition)的Preferred Leader？
(topic, partition)的AR中的第一个元素就是Preferred Leader。

### 如果首选的副本不在ISR中会发生什么?
Broker1: Leader(3), AR(1, 2, 3), ISR(2, 3)
如果这样的情况是普遍情况，会导致Broker一定程度得数据倾斜。

### 如果我指定了一个offset，Kafka怎么查找到对应的消息？

### 如果我指定了一个timestamp，Kafka怎么查找到对应的消息？

### Kafka的延时操作是怎么实现的？

### Kafka中的Exactly-Once是怎么实现的？
目前理解：每一个Producer配置一个producerId，对于同一个Producer，每一条发送的消息有一个自增的序列号，如果Broker收到同一个Producer发送了相同的两个序列号，那么就要把相同的那个序列号丢弃掉。

// now that we have valid records, offsets assigned, and timestamps updated, we need to
// validate the idempotent/transactional state of the producers and collect some metadata
val (updatedProducers, completedTxns, maybeDuplicate) = analyzeAndValidateProducerState(validRecords, isFromClient)
maybeDuplicate.foreach { duplicate =>
  appendInfo.firstOffset = duplicate.firstOffset
  appendInfo.lastOffset = duplicate.lastOffset
  appendInfo.logAppendTime = duplicate.timestamp
  appendInfo.logStartOffset = logStartOffset
  return appendInfo
}

### Kafka中的事务是怎么实现的？

### Kafka如果需要拉取历史任务，怎么样解决Broker高Load问题？
Kafka本身有没有限流机制？

### Kafka的consumer的心跳线程做什么事情？只是检测存活么？

### Kafka vagrant搭建
vagrant box add \
https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cloud-images/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box \
--name ubuntu/trusty64

### __consumer_offsets这个topic有什么特点？
根据理解，说的越多越好
1. compaction
2. retention
```
createTopic(topic, config.offsetsTopicPartitions, config.offsetsTopicReplicationFactor.toInt,
            groupCoordinator.offsetsTopicConfigs)

def offsetsTopicConfigs: Properties = {
  val props = new Properties
  props.put(LogConfig.CleanupPolicyProp, LogConfig.Compact)
  props.put(LogConfig.SegmentBytesProp, offsetConfig.offsetsTopicSegmentBytes.toString)
  props.put(LogConfig.CompressionTypeProp, ProducerCompressionCodec.name)
  props
}
```


### kafka的消费者Offset是KV Map形式的，是怎么被维护在以List形式的__consumer_offsets上？
https://www.jianshu.com/p/833b64e141f8

### kafka在恢复kv结构的offset时，真的需要从头到尾进行遍历么？
while (currOffset < highWaterMark && !shuttingDown.get()) {
这一段是什么意思？

### 消费者默认的消息事务隔离级别是什么？
我怎么记得是 read-uncommitted
```
[2019-10-08 10:44:54,021] DEBUG [Consumer clientId=consumer-1, groupId=consumerGroup1] Added READ_UNCOMMITTED fetch request for partition krp-0 at offset 10 to node 172.16.115.163:9092 (id: 2 rack: null) (org.apache.kafka.clients.consumer.internals.Fetcher)
```
这个只针对事务消息，对一般的消息无效。

### Kafka有没有类似于RocketMQ的ZeroCopy？
是用到了，但是在哪里用到的？这里写得很清楚。
https://www.jianshu.com/p/d47de3d6d8ac

sendfile不是posix规范的api，要使用sendfile，在各个平台没有统一的规定，甚至连引入的头文件都没有做规范。
FileChannelImpl.c

#if defined(__linux__) || defined(__solaris__)
#include <sys/sendfile.h>
#elif defined(_AIX)
#include <sys/socket.h>
#elif defined(_ALLBSD_SOURCE)
#include <sys/socket.h>
#include <sys/uio.h>
#define lseek64 lseek
#define mmap64 mmap
#endif

### 原先的GroupCoordinator宕机了，kafka是怎么检测到并且将GroupCoordinator转移到另一台机器，然后恢复__consumer_offsets数据的？
https://www.jianshu.com/p/833b64e141f8
按照这个文章，好像说是在收到了 LeaderAndIsr的请求后？？？

### 什么时候 「谁」 会调用 「谁」 的ApiKeys.LEADER_AND_ISR?
#### 场景1:ISR变动
在每个Partition(.scala)中，在每次expandIsr或者shrinkIsr时，都需要在zk(/isr_change_notification)上记录，以此来达到事件传播(propagate)的效果。
```
def maybeExpandIsr(replicaId: Int, logReadResult: LogReadResult): Boolean = {
  inWriteLock(leaderIsrUpdateLock) {
    // check if this replica needs to be added to the ISR
    leaderReplicaIfLocal match {
      case Some(leaderReplica) =>
          ...
          // update ISR in ZK and cache
          updateIsr(newInSyncReplicas)
          ...
        }
        ...
      case None => false // nothing to do if no longer leader
    }
  }
}

def maybeShrinkIsr(replicaMaxLagTimeMs: Long) {
  val leaderHWIncremented = inWriteLock(leaderIsrUpdateLock) {
    leaderReplicaIfLocal match {
      case Some(leaderReplica) =>
        val outOfSyncReplicas = getOutOfSyncReplicas(leaderReplica, replicaMaxLagTimeMs)
        if(outOfSyncReplicas.nonEmpty) {
          ...
          // update ISR in zk and in cache
          updateIsr(newInSyncReplicas)
          ...
        }

      case None => false // do nothing if no longer leader
    }
  }

  // some delayed operations may be unblocked after HW changed
  if (leaderHWIncremented)
    tryCompleteDelayedRequests()
}

def recordIsrChange(topicPartition: TopicPartition) {
  isrChangeSet synchronized {
    isrChangeSet += topicPartition
    lastIsrChangeMs.set(System.currentTimeMillis())
  }
}

def maybePropagateIsrChanges() {
  val now = System.currentTimeMillis()
  isrChangeSet synchronized {
    if (isrChangeSet.nonEmpty &&
      (lastIsrChangeMs.get() + ReplicaManager.IsrChangePropagationBlackOut < now ||
        lastIsrPropagationMs.get() + ReplicaManager.IsrChangePropagationInterval < now)) {
      ReplicationUtils.propagateIsrChanges(zkUtils, isrChangeSet)
      isrChangeSet.clear()
      lastIsrPropagationMs.set(now)
    }
  }
}
```
controller在onControllerFailover的registerIsrChangeNotificationListener对上面的地址进行了监听。
```
private def registerIsrChangeNotificationListener() = {
  debug("Registering IsrChangeNotificationListener")
  zkUtils.subscribeChildChanges(ZkUtils.IsrChangeNotificationPath, isrChangeNotificationListener)
}
```
controller根据leaderAndIsr的内容分享向相关的所有的broker发送LeaderAndIsr请求
```
case class IsrChangeNotification(sequenceNumbers: Seq[String]) extends ControllerEvent {

  def state = ControllerState.IsrChange

  override def process(): Unit = {
    if (!isActive) return
    try {
      val topicAndPartitions = sequenceNumbers.flatMap(getTopicAndPartition).toSet
      if (topicAndPartitions.nonEmpty) {
        updateLeaderAndIsrCache(topicAndPartitions)
        processUpdateNotifications(topicAndPartitions)
      }
    } finally {
      // delete the notifications
      sequenceNumbers.map(x => controllerContext.zkUtils.deletePath(ZkUtils.IsrChangeNotificationPath + "/" + x))
    }
  }

  ...
}
```

收到了controller发来的LeaderAndIsr请求，每个broker根据自身情况，让自己的ReplicaManager调用asLeader或者asFollower的响应
```
def becomeLeaderOrFollower(correlationId: Int,
                            leaderAndISRRequest: LeaderAndIsrRequest,
                            onLeadershipChange: (Iterable[Partition], Iterable[Partition]) => Unit): BecomeLeaderOrFollowerResult = {
  leaderAndISRRequest.partitionStates.asScala.foreach { case (topicPartition, stateInfo) =>
    stateChangeLogger.trace(s"Received LeaderAndIsr request $stateInfo " +
      s"correlation id $correlationId from controller ${leaderAndISRRequest.controllerId} " +
      s"epoch ${leaderAndISRRequest.controllerEpoch} for partition $topicPartition")
  }
  replicaStateChangeLock synchronized {
    ...
    val partitionsBecomeLeader = if (partitionsTobeLeader.nonEmpty)
      makeLeaders(controllerId, controllerEpoch, partitionsTobeLeader, correlationId, responseMap)
    else
      Set.empty[Partition]
    val partitionsBecomeFollower = if (partitionsToBeFollower.nonEmpty)
      makeFollowers(controllerId, controllerEpoch, partitionsToBeFollower, correlationId, responseMap)
    else
      Set.empty[Partition]
    ...
  }
}
```
#### 场景2:Leader变动
在kafka中，存活的broker会在zk的session中保存(/brokers/ids)地址
controller在onControllerFailover的registerBrokerChangeListener对上面的地址进行了监听。
一旦有broker无法保持与zk的session，controller会找出与这台broker相关的，但是目前已经没有了leader的(topic, partition)。controller分别用状态机重置这些partition的状态。
1. 首先将这些(topic, partition)的状态统一设置为`OfflinePartition`
2. 然后将(topic, partition)的状态从`OfflinePartition`设置为`OnlinePartition`，这个过程会触发(topic, partition)的LeaderElection
3. 最后再向相关的broker发送leaderAndIsr请求
```
// trigger OfflinePartition state for all partitions whose current leader is one amongst the newOfflineReplicas
partitionStateMachine.handleStateChanges(partitionsWithoutLeader, OfflinePartition)
// trigger OnlinePartition state changes for offline or new partitions
// partition状态机从OfflinePartition->OnlinePartition过程会触发LeaderElection
partitionStateMachine.triggerOnlinePartitionStateChange()
// trigger OfflineReplica state change for those newly offline replicas
replicaStateMachine.handleStateChanges(newOfflineReplicasNotForDeletion, OfflineReplica)
```

### Kafka中有哪些Coordinator?

### Kafka怎么做集群迁移？

### Kafka怎么做集群扩容？

### Kafka怎么Topic迁移？

### Kafka在数据倾斜的时候做数据平衡？

### 集群中的哪一台机器会成为某一个Consumer Group的GroupCoordinator？
// get metadata (and create the topic if necessary)
val (partition, topicMetadata) = findCoordinatorRequest.coordinatorType match {
  case FindCoordinatorRequest.CoordinatorType.GROUP =>
    val partition = groupCoordinator.partitionFor(findCoordinatorRequest.coordinatorKey)
    val metadata = getOrCreateInternalTopic(GROUP_METADATA_TOPIC_NAME, request.context.listenerName)
    (partition, metadata)

### Varint怎么能减少数据传输量？
http://www.zdingke.com/2018/03/17/kafka/?falerm=x2psz2
https://blog.csdn.net/u013256816/article/details/80300272
```
/**
 * Read an integer stored in variable-length format using zig-zag decoding from
 * <a href="http://code.google.com/apis/protocolbuffers/docs/encoding.html"> Google Protocol Buffers</a>.
 *
 * @param buffer The buffer to read from
 * @return The integer read
 *
 * @throws IllegalArgumentException if variable-length value does not terminate after 5 bytes have been read
 */
public static int readVarint(ByteBuffer buffer) {
    int value = 0;
    int i = 0;
    int b;
    while (((b = buffer.get()) & 0x80) != 0) {
        value |= (b & 0x7f) << i;
        i += 7;
        if (i > 28)
            throw illegalVarintException(value);
    }
    value |= b << i;
    return (value >>> 1) ^ -(value & 1);
}

/**
 * Write the given integer following the variable-length zig-zag encoding from
 * <a href="http://code.google.com/apis/protocolbuffers/docs/encoding.html"> Google Protocol Buffers</a>
 * into the buffer.
 *
 * @param value The value to write
 * @param buffer The output to write to
 */
public static void writeVarint(int value, ByteBuffer buffer) {
    int v = (value << 1) ^ (value >> 31);
    while ((v & 0xffffff80) != 0L) {
        byte b = (byte) ((v & 0x7f) | 0x80);
        buffer.put(b);
        v >>>= 7;
    }
    buffer.put((byte) v);
}
```

### Kafka Rebalance 的时机是什么时候？ new KafkaConsumer() 后 还是 consumer.subscribe() 后?
```
在调用poll()中才开始Rebalance

if (needRejoin()) {
    // due to a race condition between the initial metadata fetch and the initial rebalance,
    // we need to ensure that the metadata is fresh before joining initially. This ensures
    // that we have matched the pattern against the cluster's topics at least once before joining.
    if (subscriptions.hasPatternSubscription())
        client.ensureFreshMetadata();

    ensureActiveGroup();
    now = time.milliseconds();
}
```

### 如果某一ConsumerGroup的Consumer的数量发生变化，剩余的Consumer是如何检测到需要Rebalance了？
Rebalance的触发是在consumer的poll操作中
是不是这段？
```
@Override
public boolean needRejoin() {
    if (!subscriptions.partitionsAutoAssigned())
        return false;

    // we need to rejoin if we performed the assignment and metadata has changed
    if (assignmentSnapshot != null && !assignmentSnapshot.equals(metadataSnapshot))
        return true;

    // we need to join if our subscription has changed since the last join
    if (joinedSubscription != null && !joinedSubscription.equals(subscriptions.subscription()))
        return true;

    return super.needRejoin();
}
```

### Kafka消费者Rebalance过程
消费者查找GroupCoordinator
消费者主动向GroupCoordinator发起JOIN
作为GroupCoordinator的Broker接受到JOIN，选出一个Leader和epoch，给JOIN的成员发送响应
下面应该怎么弄啊？？？？写不下去了


这种方式的好处是，Rebalance的算法可以放在客户端，而不需要放在Broker端。


```
[2019-10-09 16:29:26,866] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Kafka consumer initialized (org.apache.kafka.clients.consumer.KafkaConsumer)
[2019-10-09 16:29:26,867] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Subscribed to topic(s): hermes.demo.pandc (org.apache.kafka.clients.consumer.KafkaConsumer)
[2019-10-09 16:29:26,867] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Sending FindCoordinator request to broker localhost:9091 (id: -1 rack: null) (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
[2019-10-09 16:29:27,026] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Initiating connection to node localhost:9091 (id: -1 rack: null) (org.apache.kafka.clients.NetworkClient)
...
[2019-10-09 16:29:27,135] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Received FindCoordinator response ClientResponse(receivedTimeMs=1570609767134, latencyMs=118, disconnected=false, requestHeader=RequestHeader(apiKey=FIND_COORDINATOR, apiVersion=1, clientId=consumer-1, correlationId=0), responseBody=FindCoordinatorResponse(throttleTimeMs=0, errorMessage='null', error=NONE, node=172.16.115.163:9092 (id: 2 rack: null))) (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
[2019-10-09 16:29:27,135] INFO [Consumer clientId=consumer-1, groupId=cg1] Discovered group coordinator 172.16.115.163:9092 (id: 2147483645 rack: null) (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
[2019-10-09 16:29:27,135] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Initiating connection to node 172.16.115.163:9092 (id: 2147483645 rack: null) (org.apache.kafka.clients.NetworkClient)
[2019-10-09 16:29:27,137] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Heartbeat thread started (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
[2019-10-09 16:29:27,137] INFO [Consumer clientId=consumer-1, groupId=cg1] Revoking previously assigned partitions [] (org.apache.kafka.clients.consumer.internals.ConsumerCoordinator)
[2019-10-09 16:29:27,138] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Disabling heartbeat thread (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
[2019-10-09 16:29:27,138] INFO [Consumer clientId=consumer-1, groupId=cg1] (Re-)joining group (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
[2019-10-09 16:29:27,141] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Sending JoinGroup ((type: JoinGroupRequest, groupId=cg1, sessionTimeout=10000, rebalanceTimeout=300000, memberId=, protocolType=consumer, groupProtocols=org.apache.kafka.common.requests.JoinGroupRequest$ProtocolMetadata@23fe1d71)) to coordinator 172.16.115.163:9092 (id: 2147483645 rack: null) (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
...
[2019-10-09 16:29:27,150] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Received successful JoinGroup response: org.apache.kafka.common.requests.JoinGroupResponse@7403c468 (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
[2019-10-09 16:29:27,150] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Performing assignment using strategy range with subscriptions {consumer-1-cce37b28-bff5-471b-84b8-c0d8f2316bdd=Subscription(topics=[hermes.demo.pandc])} (org.apache.kafka.clients.consumer.internals.ConsumerCoordinator)
[2019-10-09 16:29:27,151] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Finished assignment for group: {consumer-1-cce37b28-bff5-471b-84b8-c0d8f2316bdd=Assignment(partitions=[hermes.demo.pandc-0, hermes.demo.pandc-1])} (org.apache.kafka.clients.consumer.internals.ConsumerCoordinator)
[2019-10-09 16:29:27,152] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Sending leader SyncGroup to coordinator 172.16.115.163:9092 (id: 2147483645 rack: null): (type=SyncGroupRequest, groupId=cg1, generationId=13, memberId=consumer-1-cce37b28-bff5-471b-84b8-c0d8f2316bdd, groupAssignment=consumer-1-cce37b28-bff5-471b-84b8-c0d8f2316bdd) (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
[2019-10-09 16:29:27,156] INFO [Consumer clientId=consumer-1, groupId=cg1] Successfully joined group with generation 13 (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
[2019-10-09 16:29:27,156] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Enabling heartbeat thread (org.apache.kafka.clients.consumer.internals.AbstractCoordinator)
[2019-10-09 16:29:27,157] INFO [Consumer clientId=consumer-1, groupId=cg1] Setting newly assigned partitions [hermes.demo.pandc-1, hermes.demo.pandc-0] (org.apache.kafka.clients.consumer.internals.ConsumerCoordinator)
[2019-10-09 16:29:27,157] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Fetching committed offsets for partitions: [hermes.demo.pandc-1, hermes.demo.pandc-0] (org.apache.kafka.clients.consumer.internals.ConsumerCoordinator)
[2019-10-09 16:29:27,159] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Resetting offset for partition hermes.demo.pandc-1 to the committed offset 5 (org.apache.kafka.clients.consumer.internals.Fetcher)
[2019-10-09 16:29:27,159] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Resetting offset for partition hermes.demo.pandc-0 to the committed offset 9 (org.apache.kafka.clients.consumer.internals.Fetcher)
[2019-10-09 16:29:27,160] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Added READ_UNCOMMITTED fetch request for partition hermes.demo.pandc-1 at offset 5 to node 172.16.115.163:9091 (id: 1 rack: null) (org.apache.kafka.clients.consumer.internals.Fetcher)
[2019-10-09 16:29:27,160] DEBUG [Consumer clientId=consumer-1, groupId=cg1] Added READ_UNCOMMITTED fetch request for partition hermes.demo.pandc-0 at offset 9 to node 172.16.115.163:9091 (id: 1 rack: null) (org.apache.kafka.clients.consumer.internals.Fetcher)
```


### 异步发送后，回调是否会按照发送的真实顺序进行回调？
我的印象是可以的


### Kafka是否支持Long Polling
https://www.jianshu.com/p/34dc83e90f98
支持：
/**
 * <code>fetch.max.wait.ms</code>
 */
public static final String FETCH_MAX_WAIT_MS_CONFIG = "fetch.max.wait.ms";
private static final String FETCH_MAX_WAIT_MS_DOC = "The maximum amount of time the server will block before answering the fetch request if there isn't sufficient data to immediately satisfy the requirement given by fetch.min.bytes.";



// probably unblock some follower fetch requests since log end offset has been updated
replicaManager.tryCompleteDelayedFetch(TopicPartitionOperationKey(this.topic, this.partitionId))

从下面的代码来看，貌似是支持的。
kafka.cluster.Partition#appendRecordsToLeader
// some delayed operations may be unblocked after HW changed
if (leaderHWIncremented)
  tryCompleteDelayedRequests()

/**
   * Try to complete any pending requests. This should be called without holding the leaderIsrUpdateLock.
   */
  private def tryCompleteDelayedRequests() {
    val requestKey = new TopicPartitionOperationKey(topicPartition)
    replicaManager.tryCompleteDelayedFetch(requestKey)
    replicaManager.tryCompleteDelayedProduce(requestKey)
    replicaManager.tryCompleteDelayedDeleteRecords(requestKey)
  }

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