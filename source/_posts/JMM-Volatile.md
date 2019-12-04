---
title: JMM-Volatile
date: 2019-11-28 15:15:18
tags:
---

### Java中的`Volatile`关键词，到底是什么解决一个什么问题？
对于一些'宽'的数据类型，对其进行读写访问，怎么能保证读写原子性(读写原子性)，不是'++'原子性。

### 怎么写一个正确的DCL（单例）？
http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/

### ArrayBlockingQueue中的这段代码和注释是什么意思？
lock.lock(); // Lock only for visibility, not mutual exclusion
```
public ArrayBlockingQueue(int capacity, boolean fair,
                            Collection<? extends E> c) {
    this(capacity, fair);

    final ReentrantLock lock = this.lock;
    lock.lock(); // Lock only for visibility, not mutual exclusion
    try {
        int i = 0;
        try {
            for (E e : c) {
                checkNotNull(e);
                items[i++] = e;
            }
        } catch (ArrayIndexOutOfBoundsException ex) {
            throw new IllegalArgumentException();
        }
        count = i;
        putIndex = (i == capacity) ? 0 : i;
    } finally {
        lock.unlock();
    }
}
```