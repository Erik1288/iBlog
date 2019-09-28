---
title: Flink-CheckoutPoint-Barrier
date: 2019-01-04 16:00:37
tags:
---


http://ju.outofmemory.cn/entry/370841

### 这个模型和JVM的Safepoint太像了


https://www.jianshu.com/p/9b10313fde10
果然是通了
```
At-Most-Once
At-Least-Once
Exactly-Once
End-to-end Exactly-Once

当不采用checkpoint时，每个event做多就只会被处理一次，这就是At-Most-Once。

当不开启Barrier对齐时，上图中的Source1来的在barrier后面的一些event有可能比Source2的barrier要先到Task1.1，因为我们没有cache这些event，所以他们会正常被处理并有可能更新Task1.1的state。这样，在回复checkpoint后，Task1.1的state可能就是处理了某些checkpoint“时刻”之后数据的状态。但是对于Source1来说，他还是会offset到正常的checkpoint“时刻”的位置，那么之前处理过的barrier后面的event可能还是会被再次放入这个流中。那么这些event就不能保证“只处理一次”了，有可能处理多次，这就是At-Least-once。
如果在Task1.1.处，先来的barrier后面的event都被cache了，那么就不会影响到这个task的state。那么Task1.1的checkpoint的state就能准确反映checkpoint“时刻”的情况。那么checkpoint回复后也不会有前面说的问题，这就是Exactly-Once。但是因为Exactly-Once引入了cache机制，这会给checkpoint动作带来额外的时延（latency）。
End-to-end Exactly-Once需要结合外部系统一起完成，这里就不做讨论。

作者：WoodsWalker
链接：https://www.jianshu.com/p/9b10313fde10
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
```


解读Flink中轻量级的异步快照机制--论文
https://blog.csdn.net/lmalds/article/details/54925877