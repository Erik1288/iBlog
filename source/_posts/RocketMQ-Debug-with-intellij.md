---
title: RocketMQ在Intellij中的终极调试技巧（一）
date: 2017-08-20 13:28:52
tags: RocketMQ
---

### 调试难点

如果虚拟机够多
可以规划将MQ的各个部分部署在不同机器上，并且在所有子系统启动时加上远程调试。然后Intellij创建几个Remote debug窗口。

如果没有虚拟机，只有一台Mac，接下去的内容将对RocketMQ的调试有非常大的帮助。

### 调试界面

![image.png](http://upload-images.jianshu.io/upload_images/716353-1c36c4f4f54283e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### Name Server

### Broker
Broker的调试最为麻烦，
如果在学习RocketMQ的初期，建议单启动一个Broker，减少复杂度，关注主要流程代码。
如果要深入学习和调试，要启动Master和Slave，开启主从同步功能，也是会发生端口和文件目录冲突的地方。

store的目录都为`System.getProperty("user.home") + File.separator + "store"`，
所以我们需要对Master和Slave进行分离，方法很多种，这里介绍一种，在启动参数中配置不同的`user.home`。

Port的分离可以放在不同的配置文件中：broker-a.properties，broker-a-s.properties

#### Master Broker

```
VM options:-Drocketmq.home.dir=/Users/eric/Code/middleware/incubator-rocketmq -Drocketmq.namesrv.addr=mac:9876 -Duser.home=/Users/eric/store/master
Program arguments:-c /Users/eric/Code/middleware/incubator-rocketmq/conf/2m-2s-sync/broker-a.properties

broker-a.properties:
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
listenPort=11111
```

![image.png](http://upload-images.jianshu.io/upload_images/716353-269519f3558de100.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### Slave Broker
```
VM options:-Drocketmq.home.dir=/Users/eric/Code/middleware/incubator-rocketmq -Drocketmq.namesrv.addr=mac:9876 -Duser.home=/Users/eric/store/slave
Program arguments:-c /Users/eric/Code/middleware/incubator-rocketmq/conf/2m-2s-sync/broker-a-s.properties

broker-a-s.properties:
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
listenPort=22222
```

![image.png](http://upload-images.jianshu.io/upload_images/716353-473218e8e6936674.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置完后依次启动 Name Server, Master Broker, Slave Broker， Producer

完整的Store目录结构截图

![](https://ws1.sinaimg.cn/large/006tKfTcgy1fkedpuigp8j317e1a80zy.jpg)