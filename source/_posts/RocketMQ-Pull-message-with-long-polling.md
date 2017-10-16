---
title: RocketMQ——高性能PullRequest秘籍之长轮询原理分析
date: 2017-09-19 16:58:52
tags: RocketMQ
---


分成两部分，Client和Broker

### Client: 

### Broker:
首先PullMessageProcessor用相应的线程池调用processRequest，去ConsumerQueue中找消息，
如果找到了，没有什么好说的，直接用Netty模块将数据写进相应的channel，客户端获取到了数据后进行并行消费；
如果没有消息，那么将当前的pullRequest放入PullRequestHoldService的pullRequestTable进行suspend。
``` java 
case ResponseCode.PULL_NOT_FOUND:
    if (brokerAllowSuspend && hasSuspendFlag) {
        long pollingTimeMills = suspendTimeoutMillisLong;
        if (!this.brokerController.getBrokerConfig().isLongPollingEnable()) {
            pollingTimeMills = this.brokerController.getBrokerConfig().getShortPollingTimeMills();
        }

        String topic = requestHeader.getTopic();
        long offset = requestHeader.getQueueOffset();
        int queueId = requestHeader.getQueueId();
        PullRequest pullRequest = new PullRequest(request, channel, pollingTimeMills,
            this.brokerController.getMessageStore().now(), offset, subscriptionData, messageFilter);
        this.brokerController.getPullRequestHoldService().suspendPullRequest(topic, queueId, pullRequest);
        // 此处将repsonse设置为null，remote-server将不会给对应的channel发送响应信息。那响应的信息何时发送，有两种情况：
        // 1. PullRequestHoldService hold了足够的时间后
        // 2. 有新的信息被发送至队列后
        response = null;
        break;
    }
```
那么，消息刷入CommitLog后，怎么样让这个Hold住的PullRequest感知到消息的到来？
答案是，DefaultMessageStore.ReputMessageService线程。
ReputMessageService开启时就进行了一个近实时的空循环，
``` java
while (!this.isStopped()) {
    try {
        Thread.sleep(1);
        this.doReput();
    } catch (Exception e) {
        DefaultMessageStore.log.warn(this.getServiceName() + " service has exception. ", e);
    }
}
```
检测CommitLog中的MaxOffset是否在变大，变大了说明有新的消息已经存进了CommitLog，紧接着构建一个dispatchRequest，再让DefaultMessageStore调用doDispatch(dispatchRequest)，
该方法并没有开启新的线程，一个做了几件事情，
第一，将新的消息刷入consumerQueue，最小2页，作用也非常明显，到时候要获取一个消息，consumerQueue可以用logicOffset定位到CommitLog的PhyicOffset，是一个无法或缺的索引，
第二，将新的消息写入index file用于后续更加复杂的查询，
第三，计算bitmap。当doDispatch顺利执行完后。

重点来了，之后触发messageArrivingListener的arriving方法，让pullRequestHoldService调用notifyMessageArriving，
开启新的线程再一次让PullMessageProcessor调用processRequest来处理原来的那个pullRequest，但此时由于consumerQueue已经构建好了，所以会正常获取到消息，正常用netty模块进行一个对client的应答。
``` java
// 用一个接近空轮询
private void doReput() {
    for (boolean doNext = true; this.isCommitLogAvailable() && doNext; ) {
        ...
        SelectMappedBufferResult result = DefaultMessageStore.this.commitLog.getData(reputFromOffset);
        ...
        this.reputFromOffset = result.getStartOffset();

        for (int readSize = 0; readSize < result.getSize() && doNext; ) {
            DispatchRequest dispatchRequest =
                DefaultMessageStore.this.commitLog.checkMessageAndReturnSize(result.getByteBuffer(), false, false);
            int size = dispatchRequest.getMsgSize();
            ...
            if (size > 0) {
                // dispatch到构建consumerQueue和index file的调度器中
                DefaultMessageStore.this.doDispatch(dispatchRequest);
                
                if (BrokerRole.SLAVE != DefaultMessageStore.this.getMessageStoreConfig().getBrokerRole()
                    && DefaultMessageStore.this.brokerConfig.isLongPollingEnable()) {
                    // 通知suspend pullRequest的PullRequestHoldService解除对pullRequest的hold
                    DefaultMessageStore.this.messageArrivingListener.arriving(dispatchRequest.getTopic(),
                        dispatchRequest.getQueueId(), dispatchRequest.getConsumeQueueOffset() + 1,
                        dispatchRequest.getTagsCode(), dispatchRequest.getStoreTimestamp(),
                        dispatchRequest.getBitMap(), dispatchRequest.getPropertiesMap());
                }

            } 
            ...
        }
    }
}
```

整个过程时序图

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fkeelo8oxpj30za0g2adr.jpg)
