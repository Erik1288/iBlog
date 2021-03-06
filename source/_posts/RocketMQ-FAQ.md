---
title: 面向问题的RocketMQ原理整理
date: 2017-08-09 14:42:53
tags: RocketMQ
---

> 先来一个图，总结得非常好

https://blog.csdn.net/qq_27529917/article/details/79595395
https://upload-images.jianshu.io/upload_images/6302559-48fbc4f75fbf2412.png

### 顺序消息是怎么实现的？
消息的重复消费是不可避免的，但是消息的顺序需要保证。
a b c d e -> a a b c d eee，但是不能a c b d e

### RocketMQ与Kafka架构上的巨大差异 -- CommitLog与ConsumeQueue
https://blog.csdn.net/chunlongyu/article/details/54576649
```
每个topic_partition对应一个日志文件，Producer对该日志文件进行“顺序写”，Consumer对该文件进行“顺序读”。

但正如在“拨乱反正”续篇中所提到的：这种存储方式，对于每个文件来说是顺序IO，但是当并发的读写多个partition的时候，对应多个文件的顺序IO，表现在文件系统的磁盘层面，还是随机IO。

因此出现了当partition或者topic个数过多时，Kafka的性能急剧下降

虽然每个文件是顺序IO，但topic或者partition过多，每个文件的顺序IO，表现到磁盘上，还是随机IO。
```

### RMQ的Name Server怎么动态扩展？

### RMQ的Broker怎么动态扩展？

### msgId和offsetMsgId的区别
对于客户端来说msgId是由客户端producer实例端生成的（具体来说，调用“MessageClientIDSetter.createUniqIDBuffer()”方法生成唯一的Id），offsetMsgId是由服务端Broker端在写入消息时生成的（采用“IP地址+Port端口”与“CommitLog的物理偏移量地址”做了一个字符串拼接），其中offsetMsgId就是我们在rocketMQ控制台直接输入查询的那个messageId。

### 异步刷盘，CommitLog和ConsumeQueue哪一个先落盘？
当消息放到了commitlog 的page cache（即没有刷盘）后，异步写入consumequeue。从逻辑上来讲，consumequeue的构建是依赖commitlog 的，但是由于刷盘是异步的，所以落盘的顺序是不一定的。

### MQ为什么内部要使用好多队列？

1. send性能
理论上队列入口要加锁（要么是程序的锁，要么是文件的锁）来保证同步，如果队列非常多，虽然不能把锁去掉，但可以减小并发线程对锁的竞争。这个是以前的结论，但实际情况是，consume queue只有一个线程来做。首先，commit log文件只有一个，假如要执行sendMessage，一定要用sync或者自旋锁（之后的版本为了考虑性能），消息都是顺序写进commit log的。对于一个topic的某一个consumer group，consumerQueue文件就有很多个，写consumerQueue文件由一个ReputMessageService的线程近实时空转而触发，也是单个线程。从写consumerQueue文件（send message）的角度来分析，多个队列是没有好处的。
2. consume性能
如果从读consumerQueue文件（pull message）的角度来分析，那确实是可以提高并行度。

### MQ的NameServer集群节点中间有没有同步数据？与Zookeeper集群的区别是什么？

### MQ的topic有序性怎么保证

MessageQueueSelector中的List<MessageQueue> mqs指的是所有的broker加起来的队列。

### 怎么让两个Broker中都有某个topic的信息，如果只有一个有，就没法做send的双broker负载均衡

Client发送一个新的topic，在name server上是取不到信息的，所以先用在本地放上Map来缓存，那问题是Broker什么时候给name server发送创建Topic route info的请求？在checkMsg方法中，this.brokerController.registerBrokerAll(false, true);
MQ的消费者端如果是用push的方式回调，执行listener会开启新的线程用来接收和返回（确认）ack，但一般我们会再开我们的新线程去做消费的事情，这个时候如果由于外界原因进程死亡，那么这些消息就丢失了。

### 哪些情况会引起rebalance?    

mq数量，客户端数量

### 怎么设计Netty的同步调用与异步调用？

掌握CountDownLatch和（限流）

### 万一NameServer都被重启过，那producer或者consumer调用getTopicRouteInfo，信息从何而来？

1. UpdateRouteInfo应该会上传一个新的Topic的信息。

### RocketMQ落盘是异步+定时的，还是就异步的？

异步+定时，时间默认为500ms。

Client向broker pull message 都是以一个一个queue为单位的。processQueueTable代表，这里面的queue正在源源不断向broker请求数据中。而broker的queue的数量是可能发生变化的，所以，要时常对processQueueTable进行必要的整理。

一个Subscription对应一次rebalance调用，一般的情况是n个topic+1个Retry topic。每一次rebalance调用，都会对当前的topic分配broker中的MQ队列，如果分到n个，那个生成n个PullRequest。

RebalanceServer生成 PullRequest 会调用PullMessageService 放入 PullRequest的pullRequestQueue中，PullMessageService从pullRequestQueue队列中take出PullRequest，异步调用Netty。当网络将数据传输回来时，调用PullCallback中的ConsumeMessageService，遍历所有原始消费，对每个消息，生成一个ConsumeRequest，开启多线程进行消费。消费时，返回值怎么处理？比如 Re-consume-later, success。客户端开启一个线程定时去更新Broker的consumerOffset。

### 为什么一旦有producer发送了消息，consumer会几乎无延迟得立刻收到消息？

1. consumer长轮询，在没有消息时hold在server端。
2. commit log的max offset增大，触发空轮询ReputMessageService，构建完consume queue之后，调用messageArrivingListener通知消息到了，从而重新processRequest来pull消息。
但是问题发现了，构建ConsumeQueue理论上是有一个性能极限值的，那这样的话，不是会很容易发送的消息，消费会有延时么？
```java
@Override
public void run() {
    DefaultMessageStore.log.info(this.getServiceName() + " service started");

    while (!this.isStopped()) {
        try {
            // 这里睡眠了1ms，可以算出写入ConsumerQueue的性能极限值
            Thread.sleep(1);
            this.doReput();
        } catch (Exception e) {
            DefaultMessageStore.log.warn(this.getServiceName() + " service has exception. ", e);
        }
    }

    DefaultMessageStore.log.info(this.getServiceName() + " service end");
}
```

未解决问题：

### Broker里面的protectBroker是用来做什么的？

### CommitLog和ConsumerQueue这两个文件的读写是顺序的么？效率怎么样？

### RocketMQ在哪几种情况下，可能会出现消息重复的问题？

1. 一个consume_group一开始订阅了*tag，之后加了具体的，会发生什么。

### kafka的发送消息性能非常高，常用于日志缓冲，是不是用「Oneway」的方式？不然producer发送请求，broker处理请求，producer接受响应，这三段的时间是无论如何都无法缩减的。

### RocketMQ的事务消息是什么？
看到一段描述很有趣，当发送了分布式事务消息时，如果Producer因为意外宕机，Broker会回调Producer Group的另一一台Producer来确认事务状态。
2. Commit Log 中存储了所有的元信息，包含消息体，类似于 Mysql、Oracle 的 redolog，所以只要有 Commit
   Log 在，Consume Queue 即使数据丢失，仍然可以恢复出来。怎么理解？
3. 在Broker的SendMessageProcessor中，主干线程中只做了一件事，那就是把最新的消息Append到CommitLog中，并且通知Flush线程去force() mmap。所以，构建ConsumeQueue，构建索引这些事情都是异步的，那好像也没有看到通知这些线程，究竟是在哪里通知的呢？ Dispatcher（调度器）ReputMessageService。

### Debug形式启动Producer，Producer发送消息，断点停止后，Producer发送消息会超时失败3次，但此时如果松开断点，让他执行完，那消息是在CommitLog中呢，还是不在？

### MQ在关闭的时候，CommitLog的内容和ConsumeQueue的内容需要能对上号的，但万一异常关闭导致没有对上号，应该怎么处理？CheckPoint机制有没有用？

### 一个消息已经消费过了，能在控制台上选择进行重复投递么？

可以。

### pagecache到底是个什么东西？

### 对于不可读的Consume Queue，Consumer rebalance时会不会考虑到？

如果没有考虑到，那不可读的队列也分给了Consumer，造成Consumer的浪费。看过源码了，没有问题。

### topic中有readQ, writeQ, 那topic本身的权限是用来做什么的？

### 多个namesrv不一致问题

### 如果某个topic的consume queue上有数据，那设置可读可写队列为0后，数据是不是读不到了。（我知道不会丢失）

### 写Consume queue完全是一个单线程DefaultMessageStore.ReputMessageService中，那就没有锁竞争了，一个Topic到底放多少个队列效率达到最高，是越少越好，还是越多越好？

### namesrv是无状态的，可以随意部署，但Broker启动后怎么是怎么知道新增加的namesrv的？

### 如果发送了某个topic的某个tag的消息，那订阅了该topic的consumer是有可能过滤掉该tag的信息的。那怎么样才能知道该消息是“订阅了，被过滤了”的状态呢？

首先，根据queueId和consume offset可以判断出当前消息是不是已经被

### ConsumerQueue文件删除了，能够复原么？原理

可以

### index file删除了，能够复原么？原理

可以

### CommitLog和ConsumeQueue在非正常关机下变得不一致，如何恢复？

### ReputMessageService中空转监听是Commitog的offset，当有新的消息时，先是构建consume queue，然后通知PullMessageHoldService，那整个过程在哪里对消息进行过滤，过滤用的是tag，consume queue中也有tag的hash，是不是只需要对比这两个值就好？
5. 如果一个4G的文件用mmap映射到Java的MappedByteBuffer中，是绝对不可能整体加载进内存的。一个ByteBuffer就是一个有限的byte数组，但是，我们理论上可以在这个数组的任何位置（position）对数组进行读和写，然后映射进文件。如果OS能预感到我们的文件是顺序读写的，那么内存到文件的速度会非常快，如果是随机的，OS没法预测下次的读写位置，这样速度会变慢（这部分去查询下）
6. PullConsumer：用consumer.pull(MessageQueue mq, String subExpression, long offset, int maxNums)方法去获取Broker的消息，如果第一次offset传了0，获取到了数据，第二次还是传0，还是能获取到数据，是什么原因？Broker的consumerOffset.json为什么不起作用？Pull和Push消费到的点什么时候会被persist到config/consumerOffset.json?详见MQClientInstance.persistAllConsumerOffset会把offsetTable定时发到Broker。

### 程序中处理MappedByteBuffer要特别注意些什么？

A MemoryMappedBuffer has a fixed size, but the file it’s mapped to is elastic.所以一旦当前的程序（或者说当前的线程）对文件的内容失去了独占权，比如文件的内容被其它的线程改了，那原先的线程访问MappedByteBuffer对应位置的缓存就会抛出异常，所以，当我们要用MappedByteBuffer时，一定要确保在多线程下是互斥的。
这个理解是不对的！！！！

### RocketMQ发送，接受消息的性能与队列数量的关系。

可以查看kafka相关的信息。

### 如果让你设计一个消息中间件？你会怎么设计？

### 如果一个消息没有消费成功，会隔1s,5s,ns重新消费，这个是怎么实现的？

### 构建IndexService是否可以做成异步？

因为它如果卡住，会影响长轮询，从而影响消息接收的实时性。

### Netty线程问题
netty在触发channelRead0的时候，所用的线程是不是workEventLoopGroup所指定的线程
nettyClientWorkerThread

### netty eventLoopGroup 是什么？

### 网络端口映射
Server:8080
curl 'localhost:8080'
lsof -i:8080

### 待解决问题
```
org.apache.rocketmq.client.exception.MQClientException: No route info of this topic, binlog-msg
See http://rocketmq.apache.org/docs/faq/ for further details.
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendDefaultImpl(DefaultMQProducerImpl.java:537)
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:1038)
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:996)
	at org.apache.rocketmq.client.producer.DefaultMQProducer.send(DefaultMQProducer.java:212)
	at org.apache.rock
```

### 问题
2017-10-11 07:21:38 WARN SendMessageThread_1 - Offset for /root/store/commitlog/00000000002147483648 not matched. Request offset: 4051326754958557513, index: -521875234, mappedFileSize: 1073741824, mappedFiles count: 1
2017-10-11 07:21:38 WARN SendMessageThread_1 - findMappedFileByOffset failure. 
java.lang.ArrayIndexOutOfBoundsException: -521875234

请教：rocketmq的这个异常应该如何解决呢？


### RocketMQ水位线设置

非常重要

### RocketMQ 业务KEY相同导致哈希冲突

### RocketMQ的Commit log什么时候munmap

### RocketMQ协议

RocketMQ LengthBasedField

### MQ系统限流和Flow Control

### RocketMQ的NameServer是不是支持分布式一致性

### 现在这个版本，当Master挂掉时，Slave可以升级成新的Master么


### RocketMQ怎么样使用性能会非常差？

刷盘的策略，linux 配置的脏页的数量。脏页的比例，和脏页的大小

在公司遇到一种情况，就是不断消费失败

### RocketMQ到底用的是poll,epoll,还是其它的东西？？？？？？
http://www.jianshu.com/p/7835726dc78b?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation

http://www.jianshu.com/p/5ab57182af89?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation

### 同步刷盘会不会丢消息?

### 为什么kafka没有commitlog文件，而rocketmq有?

### kafka什么场景下，性能会比较低？

### 为什么有了kafka，阿里还要开发出rocketmq，rocketmq解决了什么kafka的问题？

### RocketMQ怎么通过Pagecache加速的？
![image.png](http://processon.com/chart_image/5b892eb9e4b0bd4db926c7a4.png?=1)

### 怎么保证同步刷盘也有效率？
发送消息的并发线程非常多，但是CommitLog只有一个，要想顺序写入内存然后刷盘必须上锁串行化（串行化后，磁盘IO竞争小）。串行排队等待刷盘的消息非常非常多，肯定要进行批处理，GroupCommit就是这个思想。GroupCommit首先处理队列A那些已经轮到的消息，当等待排队的刷盘的消息过来后，先把它们放到队列B中，等A中的所有消息都持久化后，锁住队列B，不让消息再进来，处理队列B，让刚刚空的队列A去接受之后排队的消息。

批量一次性全部刷盘

### RocketMQ QueueSelector怎么应对Topic的ConsumeQueue扩容？
```
Although it’s possible to increase the number of partitions over time, one has to be careful if messages are produced with keys. When publishing a keyed message, Kafka deterministically maps the message to a partition based on the hash of the key. This provides a guarantee that messages with the same key are always routed to the same partition. This guarantee can be important for certain applications since messages within a partition are always delivered in order to the consumer. If the number of partitions changes, such a guarantee may no longer hold. To avoid this situation, a common practice is to over-partition a bit.
```

### 消费情况
producerGroup只是往topic发送消息。consumerGroup只是消费topic。在broker端，消费的key为topic@consumerGroup，消费的offset取决于consumer设置的，CONSUME_FROM_LAST_OFFSET，CONSUME_FROM_FIRST_OFFSET,CONSUME_FROM_TIMESTAMP
.CONSUME_FROM_TIMESTAMP在实际应用中还是比较少的使用。这三个区别是什么呢？
1. 假如consumer配置为cluster模式，相同的consumerGroup，且设置客户端消费offset为CONSUME_FROM_FIRST_OFFSET，
假如topic T的队列1的maxoffset为100，那么consumerA在启动后的队列1的消费是从0消费到100，consumerB在启动后的队列1也会从0消费到100。后续的消息即要么consumerA，要么consumerB消费。
如果客户端设置的offset为CONSUME_FROM_LAST_OFFSET，那么consumerB在启动后的队列1是从100开始消费。后续消息消费方式相同。
CONSUME_FROM_TIMESTAMP表示从0到100之间的某个时间点后开始消费。
2. 假如consumer配置为cluster模式，不同的consumerGroup，这种情况的消费和广播模式一样。也就是说，后续的消息，不同的组都会收到。唯一区别在于，广播模式把已消费的进度信息保存在consumer的机器上，而集群模式保存在broker上。