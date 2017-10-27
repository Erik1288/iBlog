---
title: 面向问题的RocketMQ原理整理
date: 2017-08-09 14:42:53
tags: RocketMQ
---

工作和学习中使用RocketMQ产生的困惑和总结，属于临时半成品文章，整理完了会单独成文。

#### 为什么RocketMQ性能很高？

主要是消息存储的方式影响着性能。

#### MQ为什么内部要使用好多队列？

理论上队列入口要加锁（要么是程序的锁，要么是文件的锁）来保证同步，如果队列非常多，虽然不能把锁去掉，但可以减小并发线程对锁的竞争。这个是以前的结论，但实际情况是，consume queue只有一个线程来做。首先，commit log文件只有一个，假如要执行sendMessage，一定要用sync或者自旋锁（之后的版本为了考虑性能），消息都是顺序写进commit log的。对于一个topic的某一个consumer group，consumerQueue文件就有很多个，写consumerQueue文件由一个ReputMessageService的线程近实时空转而触发，也是单个线程。从写consumerQueue文件（send message）的角度来分析，多个队列是没有好处的。如果从读consumerQueue文件（pull message）的角度来分析。

#### MQ的NameServer集群节点中间有没有同步数据？与Zookeeper集群的区别是什么？

#### MQ的topic有序性怎么保证

MessageQueueSelector中的List<MessageQueue> mqs指的是所有的broker加起来的队列。

#### 怎么让两个Broker中都有某个topic的信息，如果只有一个有，就没法做send的双broker负载均衡

Client发送一个新的topic，在name server上是取不到信息的，所以先用在本地放上Map来缓存，那问题是Broker什么时候给name server发送创建Topic route info的请求？在checkMsg方法中，this.brokerController.registerBrokerAll(false, true);
MQ的消费者端如果是用push的方式回调，执行listener会开启新的线程用来接收和返回（确认）ack，但一般我们会再开我们的新线程去做消费的事情，这个时候如果由于外界原因进程死亡，那么这些消息就丢失了。

#### 哪些情况会引起rebalance?    

mq数量，客户端数量

#### 怎么设计Netty的同步调用与异步调用？

掌握CountDownLatch和（限流）

#### 万一NameServer都被重启过，那producer或者consumer调用getTopicRouteInfo，信息从何而来？

1. UpdateRouteInfo应该会上传一个新的Topic的信息。

#### RocketMQ落盘是异步+定时的，还是就异步的？

异步+定时，时间默认为500ms。

Client向broker pull message 都是以一个一个queue为单位的。processQueueTable代表，这里面的queue正在源源不断向broker请求数据中。而broker的queue的数量是可能发生变化的，所以，要时常对processQueueTable进行必要的整理。

一个Subscription对应一次rebalance调用，一般的情况是n个topic+1个Retry topic。每一次rebalance调用，都会对当前的topic分配broker中的MQ队列，如果分到n个，那个生成n个PullRequest。

RebalanceServer生成 PullRequest 会调用PullMessageService 放入 PullRequest的pullRequestQueue中，PullMessageService从pullRequestQueue队列中take出PullRequest，异步调用Netty。当网络将数据传输回来时，调用PullCallback中的ConsumeMessageService，遍历所有原始消费，对每个消息，生成一个ConsumeRequest，开启多线程进行消费。消费时，返回值怎么处理？比如 Re-consume-later, success。客户端开启一个线程定时去更新Broker的consumerOffset。

#### 为什么一旦有producer发送了消息，consumer会几乎无延迟得立刻收到消息？

首先网络层，和基于事件编程的Netty有关。跟网络有关，网络收到数据后，立刻进入回调代码。ReputService空轮询有关。

技巧

1. 充分利用作者写好的单元测试，对原理的掌握会有帮助
2. RocketMQ有太多的事情是用 （短时间）定时+唤醒 的方式异步执行的，想要更好得了解原理，最好把定时的时间改得大一点，这样多线程的调试会好做很多。

未解决问题：

#### Broker里面的protectBroker是用来做什么的？

#### CommitLog和ConsumerQueue这两个文件的读写是顺序的么？效率怎么样？

#### RocketMQ在哪几种情况下，可能会出现消息重复的问题？

1. 一个consume_group一开始订阅了*tag，之后加了具体的，会发生什么。

#### afka的发送消息性能非常高，常用于日志缓冲，是不是用「Oneway」的方式？不然producer发送请求，broker处理请求，producer接受响应，这三段的时间是无论如何都无法缩减的。

#### RocketMQ的事务消息是什么？
看到一段描述很有趣，当发送了分布式事务消息时，如果Producer因为意外宕机，Broker会回调Producer Group的另一一台Producer来确认事务状态。
2. Commit Log 中存储了所有的元信息，包含消息体，类似于 Mysql、Oracle 的 redolog，所以只要有 Commit
   Log 在，Consume Queue 即使数据丢失，仍然可以恢复出来。怎么理解？
3. 在Broker的SendMessageProcessor中，主干线程中只做了一件事，那就是把最新的消息Append到CommitLog中，并且通知Flush线程去force() mmap。所以，构建ConsumeQueue，构建索引这些事情都是异步的，那好像也没有看到通知这些线程，究竟是在哪里通知的呢？ Dispatcher（调度器）ReputMessageService。

#### Debug形式启动Producer，Producer发送消息，断点停止后，Producer发送消息会超时失败3次，但此时如果松开断点，让他执行完，那消息是在CommitLog中呢，还是不在？

#### MQ在关闭的时候，CommitLog的内容和ConsumeQueue的内容需要能对上号的，但万一异常关闭导致没有对上号，应该怎么处理？CheckPoint机制有没有用？

#### 一个消息已经消费过了，能在控制台上选择进行重复投递么？

可以。

#### Page Cache到底是个什么东西？

#### 对于不可读的Consume Queue，Consumer rebalance时会不会考虑到？

如果没有考虑到，那不可读的队列也分给了Consumer，造成Consumer的浪费。看过源码了，没有问题。

#### topic中有readQ, writeQ, 那topic本身的权限是用来做什么的？

#### 多个namesrv不一致问题

#### 如果某个topic的consume queue上有数据，那设置可读可写队列为0后，数据是不是读不到了。（我知道不会丢失）

#### 写Consume queue完全是一个单线程DefaultMessageStore.ReputMessageService中，那就没有锁竞争了，一个Topic到底放多少个队列效率达到最高，是越少越好，还是越多越好？

#### namesrv是无状态的，可以随意部署，但Broker启动后怎么是怎么知道新增加的namesrv的？

#### 如果发送了某个topic的某个tag的消息，那订阅了该topic的consumer是有可能过滤掉该tag的信息的。那怎么样才能知道该消息是“订阅了，被过滤了”的状态呢？

#### ConsumerQueue文件删除了，能够复原么？原理

#### index file删除了，能够复原么？原理

#### ReputMessageService中空转监听是Commitog的offset，当有新的消息时，先是构建consume queue，然后通知PullMessageHoldService，那整个过程在哪里对消息进行过滤，过滤用的是tag，consume queue中也有tag的hash，是不是只需要对比这两个值就好？
5. 如果一个4G的文件用mmap映射到Java的MappedByteBuffer中，是绝对不可能整体加载进内存的。一个ByteBuffer就是一个有限的byte数组，但是，我们理论上可以在这个数组的任何位置（position）对数组进行读和写，然后映射进文件。如果OS能预感到我们的文件是顺序读写的，那么内存到文件的速度会非常快，如果是随机的，OS没法预测下次的读写位置，这样速度会变慢（这部分去查询下）
6. PullConsumer：用consumer.pull(MessageQueue mq, String subExpression, long offset, int maxNums)方法去获取Broker的消息，如果第一次offset传了0，获取到了数据，第二次还是传0，还是能获取到数据，是什么原因？Broker的consumerOffset.json为什么不起作用？Pull和Push消费到的点什么时候会被persist到config/consumerOffset.json?详见MQClientInstance.persistAllConsumerOffset会把offsetTable定时发到Broker。

#### 程序中处理MappedByteBuffer要特别注意些什么？

A MemoryMappedBuffer has a fixed size, but the file it’s mapped to is elastic.所以一旦当前的程序（或者说当前的线程）对文件的内容失去了独占权，比如文件的内容被其它的线程改了，那原先的线程访问MappedByteBuffer对应位置的缓存就会抛出异常，所以，当我们要用MappedByteBuffer时，一定要确保在多线程下是互斥的。

#### RocketMQ发送，接受消息的性能与队列数量的关系。

可以查看kafka相关的信息。

#### 如果让你设计一个消息中间件？你会怎么设计？

#### 如果一个消息没有消费成功，会隔1s,5s,ns重新消费，这个是怎么实现的？

#### 构建IndexService是否可以做成异步？

因为它如果卡住，会影响长轮询，从而影响消息接收的实时性。

#### Netty线程问题
netty在触发channelRead0的时候，所用的线程是不是workEventLoopGroup所指定的线程
nettyClientWorkerThread

#### netty eventLoopGroup 是什么？

#### 网络端口映射
Server:8080
curl 'localhost:8080'
lsof -i:8080

#### 待解决问题
```
org.apache.rocketmq.client.exception.MQClientException: No route info of this topic, binlog-msg
See http://rocketmq.apache.org/docs/faq/ for further details.
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendDefaultImpl(DefaultMQProducerImpl.java:537)
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:1038)
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:996)
	at org.apache.rocketmq.client.producer.DefaultMQProducer.send(DefaultMQProducer.java:212)
	at org.apache.rock
```

#### 问题
2017-10-11 07:21:38 WARN SendMessageThread_1 - Offset for /root/store/commitlog/00000000002147483648 not matched. Request offset: 4051326754958557513, index: -521875234, mappedFileSize: 1073741824, mappedFiles count: 1
2017-10-11 07:21:38 WARN SendMessageThread_1 - findMappedFileByOffset failure. 
java.lang.ArrayIndexOutOfBoundsException: -521875234

请教：rocketmq的这个异常应该如何解决呢？


#### RocketMQ水位线设置

非常重要

#### RocketMQ 业务KEY相同导致哈希冲突

