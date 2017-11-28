---
title: curator——分布式锁的实现之一
date: 2017-09-28 16:58:05
tags: Curator
---


### MySQL实现


### Redis

http://blog.csdn.net/bolg_hero/article/details/78532920

### ZooKeeper
 RedLock


``` java
public class InterProcessMutexTest {
    public static final String LOCK_PATH = "/curator-kick-off/lock";
    private static CuratorFramework client;

    @BeforeClass
    public static void before() {
        client = CuratorFrameworkFactory.newClient(
                "127.0.0.1:2181",
                1000 * 10,
                3000, new ExponentialBackoffRetry(
                        1000,
                        3));
        client.start();
    }

    @Test
    public void test() throws Exception {
        InterProcessMutex lock = new InterProcessMutex(client, LOCK_PATH);
        boolean acquire = lock.acquire(5, TimeUnit.SECONDS);
        System.out.println(acquire);
        System.out.println("done");
        lock.release();
        System.in.read();
    }
}
```
