---
title: RocketMQ——IndexService 原理分析
date: 2017-10-12 19:29:40
tags: RocketMQ
---


#### RocmetMQ的IndexService设计原理

在RocketMQ中，IndexService底层是通过文件来存储的，所以，即使MQ的进程在中途重启过，索引的功能是不受影响的。
索引文件的路径是 `System.getProperty("user.home") + File.separator + "store"`，文件名是文件创建的时间，可以有多个，但，
在一个文件没有满的情况下，所有的topic的所有的列队的消息，全部都是顺序得存放在一个文件中的，这很重要，下面会详解。
![image.png](http://upload-images.jianshu.io/upload_images/716353-2ed415df19a3040b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在MQ源码中，用IndexFile这个类代表索引文件，对于每一个index file，大小都是固定的，即，都是设计好的。
index file在逻辑上被拆分成了3个部分，IndexHead + HashSlotPart + MsgIndexPart，
IndexHead
索引的开头，和索引的结构没有关系
HashSlotPart
hash的槽位，是索引的目录，用于定位消息索引在该文件的「MsgIndexPart」的位置，
可能有点绕，往下就会觉得很简单，每个槽位是等长的，占4 Byte，一个文件总的槽位数量也是定的，不可改变，槽位数越大，索引的消息越多。
MsgIndexPart
真实的消息索引，即，每个msgIndex段代表改消息在CommitLog上的PhyicOffset。每个msgIndex段也是等长的，占20 Byte（int + long + int + int）。
等长的MsgIndexPart可以理解成功一个有capacity的数组，为了使数组的空间不浪费，那消息就要从前往后一个一个append进去。


所以，在默认配置下，
每个索引文件的大小为 
`int fileTotalSize = IndexHeader.INDEX_HEADER_SIZE + (hashSlotNum * hashSlotSize) + (indexNum * indexSize);`

CommitLogDispatcherBuildIndex调用dispatch
``` java
public void dispatch(DispatchRequest request) {
    if (DefaultMessageStore.this.messageStoreConfig.isMessageIndexEnable()) {
        DefaultMessageStore.this.indexService.buildIndex(request);
    }
}
```
buildIndex时会构建好几个索引，topic#msgId=>msgIndex, topic#key1=>msgIndex, topic#key2=>msgIndex
``` java
public void buildIndex(DispatchRequest req) {
    IndexFile indexFile = retryGetAndCreateIndexFile();
    if (indexFile != null) {
        long endPhyOffset = indexFile.getEndPhyOffset();
        DispatchRequest msg = req;
        String topic = msg.getTopic();
        String keys = msg.getKeys();

        ...
        if (req.getUniqKey() != null) {
            // 构建UniqKey，也就是msgId的索引
            indexFile = putKey(indexFile, msg, buildKey(topic, req.getUniqKey()));
            ...
        }

        if (keys != null && keys.length() > 0) {
            String[] keyset = keys.split(MessageConst.KEY_SEPARATOR);
            for (int i = 0; i < keyset.length; i++) {
                String key = keyset[i];
                if (key.length() > 0) {
                    // 构建业务Key的索引
                    indexFile = putKey(indexFile, msg, buildKey(topic, key));
                    ...
                }
            }
        }
    } else {
        log.error("build index error, stop building index");
    }
}
```