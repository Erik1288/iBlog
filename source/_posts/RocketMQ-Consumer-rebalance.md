---
title: RocketMQ——Consumer Rebalance 原理分析
date: 2017-09-25 16:58:43
tags: RocketMQ
---


以topic为维度，开始进行Rebalance
以集群模式为例，先调用`List<String> cidAll = this.mQClientFactory.findConsumerIdList(topic, consumerGroup)`找到所有Consumer，然后根据topic路由信息，找到所有的MessageQueue。举个例子，有8个MessageQueue，4个Consumer,可以把MessageQueue理解为任务队列，4个Consumer大家一起分下任务，默认是平均分配（AllocateMessageQueueStrategy）策略，即说好，大家一人固定拉取并消费2个队列，如果队列的数量或者Consumer的数量不变，那这个关系就不要打破。可能有人要问了，这个给Consumer分配任务的工作，总得由一个老大哥（Leader）来执行吧，要么是Consumer里面有个Leader，要么是将该模块给集群协调者来做，不然怎么保证大家的分的方式都一样嘞。但RocketMQ恰恰是由每个Consumer各自都来做这个分配，秘诀在于下面两行代码
```
Collections.sort(mqAll);
Collections.sort(cidAll);
```
当大家获取的mqAll和cidAll集合内容和顺序都一样，按照AllocateMessageQueueAveragely#allocate的分配方式，每个Consumer就能获取到属于自己的任务队列。

```
private void rebalanceByTopic(final String topic, final boolean isOrder) {
    switch (messageModel) {
        case BROADCASTING: {
            Set<MessageQueue> mqSet = this.topicSubscribeInfoTable.get(topic);
            if (mqSet != null) {
                boolean changed = this.updateProcessQueueTableInRebalance(topic, mqSet, isOrder);
                if (changed) {
                    this.messageQueueChanged(topic, mqSet, mqSet);
                    log.info("messageQueueChanged {} {} {} {}",
                        consumerGroup,
                        topic,
                        mqSet,
                        mqSet);
                }
            } else {
                log.warn("doRebalance, {}, but the topic[{}] not exist.", consumerGroup, topic);
            }
            break;
        }
        case CLUSTERING: {
            Set<MessageQueue> mqSet = this.topicSubscribeInfoTable.get(topic);
            List<String> cidAll = this.mQClientFactory.findConsumerIdList(topic, consumerGroup);
            if (null == mqSet) {
                if (!topic.startsWith(MixAll.RETRY_GROUP_TOPIC_PREFIX)) {
                    log.warn("doRebalance, {}, but the topic[{}] not exist.", consumerGroup, topic);
                }
            }

            if (null == cidAll) {
                log.warn("doRebalance, {} {}, get consumer id list failed", consumerGroup, topic);
            }

            if (mqSet != null && cidAll != null) {
                List<MessageQueue> mqAll = new ArrayList<MessageQueue>();
                mqAll.addAll(mqSet);

                Collections.sort(mqAll);
                Collections.sort(cidAll);

                AllocateMessageQueueStrategy strategy = this.allocateMessageQueueStrategy;

                List<MessageQueue> allocateResult = null;
                try {
                    allocateResult = strategy.allocate(
                        this.consumerGroup,
                        this.mQClientFactory.getClientId(),
                        mqAll,
                        cidAll);
                } catch (Throwable e) {
                    log.error("AllocateMessageQueueStrategy.allocate Exception. allocateMessageQueueStrategyName={}", strategy.getName(),
                        e);
                    return;
                }

                Set<MessageQueue> allocateResultSet = new HashSet<MessageQueue>();
                if (allocateResult != null) {
                    allocateResultSet.addAll(allocateResult);
                }

                boolean changed = this.updateProcessQueueTableInRebalance(topic, allocateResultSet, isOrder);
                if (changed) {
                    log.info(
                        "rebalanced result changed. allocateMessageQueueStrategyName={}, group={}, topic={}, clientId={}, mqAllSize={}, cidAllSize={}, rebalanceResultSize={}, rebalanceResultSet={}",
                        strategy.getName(), consumerGroup, topic, this.mQClientFactory.getClientId(), mqSet.size(), cidAll.size(),
                        allocateResultSet.size(), allocateResultSet);
                    this.messageQueueChanged(topic, mqSet, allocateResultSet);
                }
            }
            break;
        }
        default:
            break;
    }
}
``` 

在确定了当前Consumer需要拉取消费的队列后，如果这个关系发生了变化，那要删除以前正在消费，但是现在不用消费的队列。比如，当前Consumer本来被分到了MessageQueue1和MessageQueue3，但是新加入两个Consumer，重新分配（ReBalance）后，当前消费者不再需要消费MessageQueue3了，那需要把这个关系给删除掉。
``` RebalanceImpl#updateProcessQueueTableInRebalance
Iterator<Entry<MessageQueue, ProcessQueue>> it = this.processQueueTable.entrySet().iterator();
while (it.hasNext()) {
    Entry<MessageQueue, ProcessQueue> next = it.next();
    MessageQueue mq = next.getKey();
    ProcessQueue pq = next.getValue();

    if (mq.getTopic().equals(topic)) {
        if (!mqSet.contains(mq)) {
            pq.setDropped(true);
            if (this.removeUnnecessaryMessageQueue(mq, pq)) {
                it.remove();
                changed = true;
                log.info("doRebalance, {}, remove unnecessary mq, {}", consumerGroup, mq);
            }
        } else if (pq.isPullExpired()) {
            switch (this.consumeType()) {
                case CONSUME_ACTIVELY:
                    break;
                case CONSUME_PASSIVELY:
                    pq.setDropped(true);
                    if (this.removeUnnecessaryMessageQueue(mq, pq)) {
                        it.remove();
                        changed = true;
                        log.error("[BUG]doRebalance, {}, remove unnecessary mq, {}, because pull is pause, so try to fixed it",
                            consumerGroup, mq);
                    }
                    break;
                default:
                    break;
            }
        }
    }
}
```

删除多余的消费关系后，当前Consumer就要开始给`每一个被分配到的MessageQueue`发送消息拉取请求（PullRequest）。
``` RebalanceImpl#updateProcessQueueTableInRebalance
List<PullRequest> pullRequestList = new ArrayList<PullRequest>();
for (MessageQueue mq : mqSet) {
    if (!this.processQueueTable.containsKey(mq)) {
        if (isOrder && !this.lock(mq)) {
            log.warn("doRebalance, {}, add a new mq failed, {}, because lock failed", consumerGroup, mq);
            continue;
        }

        this.removeDirtyOffset(mq);
        ProcessQueue pq = new ProcessQueue();
        long nextOffset = this.computePullFromWhere(mq);
        if (nextOffset >= 0) {
            ProcessQueue pre = this.processQueueTable.putIfAbsent(mq, pq);
            if (pre != null) {
                log.info("doRebalance, {}, mq already exists, {}", consumerGroup, mq);
            } else {
                log.info("doRebalance, {}, add a new mq, {}", consumerGroup, mq);
                PullRequest pullRequest = new PullRequest();
                pullRequest.setConsumerGroup(consumerGroup);
                pullRequest.setNextOffset(nextOffset);
                pullRequest.setMessageQueue(mq);
                pullRequest.setProcessQueue(pq);
                pullRequestList.add(pullRequest);
                changed = true;
            }
        } else {
            log.warn("doRebalance, {}, add new mq failed, {}", consumerGroup, mq);
        }
    }
}

this.dispatchPullRequest(pullRequestList);
```

```
public void dispatchPullRequest(List<PullRequest> pullRequestList) {
    for (PullRequest pullRequest : pullRequestList) {
        this.defaultMQPushConsumerImpl.executePullRequestImmediately(pullRequest);
        log.info("doRebalance, {}, add a new pull request {}", consumerGroup, pullRequest);
    }
}
```

以上所有的代码全部都是RebalanceService，用于管理Consumer和MessageQueue之间的对应关系，以及当两者任一一个数量非常变化时的重新均衡。消息真正拉取是由PullMessageService来处理的，注意到，它其实是一个线程。`class PullMessageService extends ServiceThread`，前面RebalanceSerivce的代码是在`class RebalanceService extends ServiceThread`Rebalance的线程中。现在是Rebalance线程要将PullRequest数据给PullMessage线程，肯定是通过阻塞队列，就是pullRequestQueue。
```
public void executePullRequestImmediately(final PullRequest pullRequest) {
    try {
        this.pullRequestQueue.put(pullRequest);
    } catch (InterruptedException e) {
        log.error("executePullRequestImmediately pullRequestQueue.put", e);
    }
}
```
随后就是从pullRequestQueue拉取pullRequest，去做真正的消息拉取。
```
@Override
public void run() {
    log.info(this.getServiceName() + " service started");

    while (!this.isStopped()) {
        try {
            PullRequest pullRequest = this.pullRequestQueue.take();
            this.pullMessage(pullRequest);
        } catch (InterruptedException ignored) {
        } catch (Exception e) {
            log.error("Pull Message Service Run Method exception", e);
        }
    }

    log.info(this.getServiceName() + " service end");
}
```
真正拉取的部分，是DefaultMQPushConsumerImpl#pullMessage方法中的实现。
``` DefaultMQPushConsumerImpl#pullMessage
    final ProcessQueue processQueue = pullRequest.getProcessQueue();
    if (processQueue.isDropped()) {
        log.info("the pull request[{}] is dropped.", pullRequest.toString());
        return;
    }
```
ProcessQueue会多次出现在正在拉取和消费的代码中，它存放的是某一个MessageQueue的`拉取后，但并未被消费的消息集合`，消息内部用用TreeMap`TreeMap<Long, MessageExt> msgTreeMap`来存储，用MessageQueueOffset来排序，以达到单队列内部严格的顺序。经过Rebalance后，当前Consumer可能不会再消费曾经被分配到的MessageQueue，自然也就无需再次消费之前没有消费完的数据。
```
long cachedMessageCount = processQueue.getMsgCount().get();
long cachedMessageSizeInMiB = processQueue.getMsgSize().get() / (1024 * 1024);

if (cachedMessageCount > this.defaultMQPushConsumer.getPullThresholdForQueue()) {
    this.executePullRequestLater(pullRequest, PULL_TIME_DELAY_MILLS_WHEN_FLOW_CONTROL);
    if ((queueFlowControlTimes++ % 1000) == 0) {
        log.warn(
            "the cached message count exceeds the threshold {}, so do flow control, minOffset={}, maxOffset={}, count={}, size={} MiB, pullRequest={}, flowControlTimes={}",
            this.defaultMQPushConsumer.getPullThresholdForQueue(), processQueue.getMsgTreeMap().firstKey(), processQueue.getMsgTreeMap().lastKey(), cachedMessageCount, cachedMessageSizeInMiB, pullRequest, queueFlowControlTimes);
    }
    return;
}

if (cachedMessageSizeInMiB > this.defaultMQPushConsumer.getPullThresholdSizeForQueue()) {
    this.executePullRequestLater(pullRequest, PULL_TIME_DELAY_MILLS_WHEN_FLOW_CONTROL);
    if ((queueFlowControlTimes++ % 1000) == 0) {
        log.warn(
            "the cached message size exceeds the threshold {} MiB, so do flow control, minOffset={}, maxOffset={}, count={}, size={} MiB, pullRequest={}, flowControlTimes={}",
            this.defaultMQPushConsumer.getPullThresholdSizeForQueue(), processQueue.getMsgTreeMap().firstKey(), processQueue.getMsgTreeMap().lastKey(), cachedMessageCount, cachedMessageSizeInMiB, pullRequest, queueFlowControlTimes);
    }
    return;
}

if (!this.consumeOrderly) {
    if (processQueue.getMaxSpan() > this.defaultMQPushConsumer.getConsumeConcurrentlyMaxSpan()) {
        this.executePullRequestLater(pullRequest, PULL_TIME_DELAY_MILLS_WHEN_FLOW_CONTROL);
        if ((queueMaxSpanFlowControlTimes++ % 1000) == 0) {
            log.warn(
                "the queue's messages, span too long, so do flow control, minOffset={}, maxOffset={}, maxSpan={}, pullRequest={}, flowControlTimes={}",
                processQueue.getMsgTreeMap().firstKey(), processQueue.getMsgTreeMap().lastKey(), processQueue.getMaxSpan(),
                pullRequest, queueMaxSpanFlowControlTimes);
        }
        return;
    }
}
```
以上描述了三种情况下，会让拉取消息限流。

之后构造了一个非常复杂的`PullCallback`，其实就是描述调用`pullAPIWrapper.pullKernelImpl`后，对于各种结果的回调（Callback）。
```
PullCallback pullCallback = new PullCallback() {
    @Override
    public void onSuccess(PullResult pullResult) {
        if (pullResult != null) {
            pullResult = DefaultMQPushConsumerImpl.this.pullAPIWrapper.processPullResult(pullRequest.getMessageQueue(), pullResult,
                subscriptionData);

            switch (pullResult.getPullStatus()) {
                case FOUND:
                    // deal with this case
                    break;
                case NO_NEW_MSG:
                    // deal with this case
                    break;
                case NO_MATCHED_MSG:
                    // deal with this case
                    break;
                case OFFSET_ILLEGAL:
                    // deal with this case
                    break;
                default:
                    break;
            }
        }
    }

    @Override
    public void onException(Throwable e) {
        // deal with this case
        DefaultMQPushConsumerImpl.this.executePullRequestLater(pullRequest, PULL_TIME_DELAY_MILLS_WHEN_EXCEPTION);
    }
};
```
