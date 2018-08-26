---
title: TCP-Protocol-Analyzer
date: 2017-12-22 09:54:47
tags: TCP
---

http://blog.csdn.net/q1007729991/article/details/70154359

https://www.jianshu.com/p/83ff61d074bf

在RocketMQ中，netty的Server端配置如下，显然，作者是有意多配置了几个TCP的参数
``` java
ServerBootstrap childHandler =
            this.serverBootstrap.group(this.eventLoopGroupBoss, this.eventLoopGroupSelector)
                .channel(useEpoll() ? EpollServerSocketChannel.class : NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG, 1024)
                .option(ChannelOption.SO_REUSEADDR, true)
                .option(ChannelOption.SO_KEEPALIVE, false)
                .childOption(ChannelOption.TCP_NODELAY, true)
                .childOption(ChannelOption.SO_SNDBUF, nettyServerConfig.getServerSocketSndBufSize())
                .childOption(ChannelOption.SO_RCVBUF, nettyServerConfig.getServerSocketRcvBufSize())
                .localAddress(new InetSocketAddress(this.nettyServerConfig.getListenPort()))
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch) throws Exception {
                        ch.pipeline().addLast(
                            defaultEventExecutorGroup,
                            new NettyEncoder(),
                            new NettyDecoder(),
                            new IdleStateHandler(0, 0, nettyServerConfig.getServerChannelMaxIdleTimeSeconds()),
                            new NettyConnectManageHandler(),
                            new NettyServerHandler());
                    }
                });
```

带着这个疑问，让我们来看看，到底这些参数在TCP中是什么意思？以及如何调节参数才能提高应用的性能。


用golang实现了一个MySQL Proxy，在做负载测试时，尝试用benchyou200连接向系统发送select * from xxx limit 10000的SQL，中途将benchyou关闭（ctrl+c），结果在Proxy端留下了大量[CLOSE-WAIT]的连接没有释放。
```
[nanxing@zerodb-proxy003 zeroproxy]$ ss |grep 9696
tcp    ESTAB      0      0      10.12.1.62:59696                10.12.1.69:mysql                
tcp    ESTAB      0      0      10.12.1.62:39696                10.12.1.67:mysql                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43016                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43094                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43232                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43164                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43170                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43464                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43622                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43490                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43416                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:42996                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43674                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43466                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43440                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43324                
tcp    ESTAB      0      0       ::ffff:10.12.1.62:9696                    ::ffff:10.1.133.201:52277                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43626                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43564                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43784                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43740                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43120                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43194                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43460                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43038                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43074                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43694                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43676                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43202                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43590                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43604                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43510                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43204                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43088                
tcp    CLOSE-WAIT 1      0       ::ffff:10.12.1.62:9696                  ::ffff:10.12.1.74:43456
```

### 服务器TIME_WAIT和CLOSE_WAIT详解和解决办法
https://www.cnblogs.com/sunxucool/p/3449068.html


### 怎么写CLOSE_WAIT
非常简单，写一个Proxy，proxy创建很多与mysql的连接，proxy将每个连接的指针放入某个不回收的容器，这时关闭mysql，查看proxy的连接，就是CLOSE_WAIT

### 为什么Proxy和MySQL连得好好的，会大量产生CLOSE-WAIT？
原因在于MySQL 的默认设置下，当一个连接的空闲时间超过8小时后，MySQL 就会断开该连接，而proxy连接池则以为该被断开的连接依然有效，其实连接已经是CLOSE-WAIT状态。在这种情况下，如果客户端代码向proxy连接池请求连接的话，连接池就会把已经失效的连接返回给客户端，客户端在使用该失效连接的时候即抛出异常。