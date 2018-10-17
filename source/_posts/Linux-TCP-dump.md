---
title: Linux-TCP-dump
date: 2018-10-16 16:07:36
tags:
---

### 问题描述
在数据库中间件中，下面的SQL执行了3010ms，但是在MySQL中，没有看到慢日志。最终定位到
`socket.connect(new InetSocketAddress(dbHostConfig.getIp(), dbHostConfig.getPort()), SOCKET_CONNECT_TIMEOUT);`
这行语句执行了3秒左右。

```
SELECT id
        FROM item
        WHERE op_time < 1539672074878
        AND platform_id = 1
        limit 0,1000
```

### dump指定网卡3306端口的所有TCP请求
tcpdump -i eth0 port 3306 -w /tmp/a.out

### 将a.out导入到wireshark

### 
点击`find a packet`，将`Display filter`修改成为`String`，将最前面的`Packet list`修改成`Packet bytes`，从而可以匹配TCP包内部的字符串。
查看上面的SQL，发现可以用时间作为关键字来进行查询。

### 列出所有stream
但是从包stream上来看，TCP的3次握手就是从14:41:17开始的，所以排除了网络抖动的可能性，那分析下来就只能是`new InetSocketAddress(dbHostConfig.getIp()`非常慢，该语句就是用于DNS解析。

