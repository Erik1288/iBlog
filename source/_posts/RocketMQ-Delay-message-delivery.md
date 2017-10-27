---
title: RocketMQ——延时消息投递原理分析
date: 2017-10-13 10:38:19
tags: RocketMQ
---

### 被动延时消费

``` java
consumer.registerMessageListener(new MessageListenerConcurrently() {
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
        ConsumeConcurrentlyContext context) {
        // 可能抛出异常
        boolean success = doConsume(msgs);
        
        if (success) {
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        } else {
            return ConsumeConcurrentlyStatus.RECONSUME_LATER;
        }
    }
});
```

### 主动延时消费

``` java

```

### 分布式事务之 Best Effort Delivery