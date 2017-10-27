---
title: Cobar——原理整理
date: 2017-10-18 20:24:04
tags: Cobar
---

FrontendConnection默认的NioHandler是FrontendAuthenticator, 当认证信息发来时，FrontendConnection会把handler.handle()任务放入一个线程池中，当前的handler是默认的认证器，如果认证成功了，FrontendAuthenticator会反过来将FrontendConnection的handler替换成FrontendCommandHandler。

Handler是由NioConnection主动触发的，在Handler处理完信息后，需要将处理完后的response回写给NioConnection, NioConnection把数据放入NioProcessor的writeQueue。

Cobar将数据库连接池的大小暴露在配置文件中，但为了性能考虑（我觉得），没有严格将数据库的连接数保持在这个范围内，假设连接池的大小为50，在高并发的SQL执行下，连接数可能会冲击到80-90，此时需要注意MySQL的max_connections这个配置，如果这个值比较小，那Cobar在并发执行SQL时创建连接，而MySQL握手包的内容可能会发生变化，但Cobar不会处理这种异常情况，导致Cobar抛出一下错误：
![img](https://ws4.sinaimg.cn/large/006tKfTcgy1fkmo3ieklej31kw0hz7il.jpg)

### Cobar怎么处理半包和拆包