---
title: Netty-AdaptiveRecvByteBufAllocator
date: 2019-02-25 18:21:32
tags:
---


The RecvByteBufAllocator that automatically increases and decreases the predicted buffer size on feed back.
It gradually increases the expected number of readable bytes if the previous read fully filled the allocated buffer. It gradually decreases the expected number of readable bytes if the read operation was not able to fill a certain amount of the allocated buffer two times consecutively. Otherwise, it keeps returning the same prediction.


避免扩容: ByteBuf的大小预估与AdaptiveRecvByteBufAllocator
ByteBuf如果一开始申请的不足，到极限时会智能的扩容，但也和Java一样，需要重新申请两倍的内存，然后把旧的内容复制过去，一听就是个很消耗的动作，因此，反正是堆外内存池，一开始还是给多一点吧。

另一个有趣的思路是Netty的自适应算法。Netty收到一个请求时，什么都不知道啊，那会申请多大的内存来接收它呢？在Bootstrap里可以配置，默认是 AdaptiveRecvByteBufAllocator，根据每一次收到的请求动态变化。

但如果一个应用有几个不同接口，请求的大小变来变去，会不会玩死它呢？好像会的。不过服务化体系里的特征都是请求小，返回大，请求包的大小变化不会太剧烈。