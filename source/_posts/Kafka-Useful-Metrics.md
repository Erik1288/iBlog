---
title: Kafka-Useful-Metrics
date: 2019-08-26 13:02:31
tags:
---


### consumer
kafka.consumer
    app-info:
        consumer-1: [consumer]级别
            commit-id
            start-time-ms 启动时间
            version 消费者版本

    consumer-coordinator-metrics:
        consumer-1: [consumer]级别

            assigned-partitions 分配到的partition数量

            commit-latency-avg 「commit请求」响应平均值
            commit-latency-max 「commit请求」响应最大值
            commit-rate 发送「commit请求」的频率
            commit-total 发送「commit请求」的总数

            heartbeat-rate 发送「heartbeat请求」的频率
            heartbeat-response-time-max 「heartbeat请求」的响应最大值
            heartbeat-total 发送「heartbeat请求」的数量
            last-heartbeat-seconds-ago 距离上一次「heartbeat请求」的发送的时间

            join-rate 发送「join请求」的频率
            join-time-avg
            join-time-max
            join-total

            sync-rate
            sync-time-avg
            sync-time-max
            sync-total

    consumer-fetch-manager-metrics:
        consumer-1: [consumer]级别

            fetch-total 发送「fetch请求」的数量

            bytes-consumed-rate 接收消息byte的速率
            bytes-consumed-total 接收消息byte数

            records-consumed-total 接收消息数量
            records-lag-max

            hello-topic: [topic]级别
                bytes-consumed-total
                records-consumed-total

    consumer-metrics:
        consumer-1: [consumer]级别
            connection-close-total
            connection-count
            connection-creation-rate
            connection-creation-total

            io-ratio
            io-time-ns-avg
            io-wait-ratio
            io-wait-time-ns-avg
            io-waittime-total
            iotime-total

            incoming-byte-rate
            incoming-byte-total

            outgoing-byte-rate
            outgoing-byte-total

            select-rate
            select-total

    consumer-node-metrics:
        consumer-1:




### producer

### broker