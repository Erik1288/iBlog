---
title: JVM-CAS-Campare-and-swap
date: 2017-11-10 19:48:55
tags: JVM
---
> 仅供学习交流,如有错误请指出,如要转载请加上出处,谢谢

### CAS

硬件指令

### 基于CAS的无所化（Lock-Free）设计

也就是所谓的乐观锁
``` java
/**
 * 自旋锁
 */
public class SpinLock {
    private AtomicBoolean canLock = new AtomicBoolean(true);

    public void lock() {
        boolean b;
        do {
            b = canLock.compareAndSet(true, false);
        } while (!b);
    }

    public void release() {
        canLock.compareAndSet(false, true);
    }
}
```

调用lock方法时，`canLock`初始为true，cas成功执行，返回修改后的值，也就是false。然后循环在while中，只要canLock没有被重置会true，cas一直是失败的。CPU被该线程长久占用着。

### ABA问题

CAS本身是没有任何问题的，是操作系统的指令。但是，当我们用CAS原理来设计无锁化的互斥机制时，就一定会产生ABA问题。

### 优化ABA

对于某些系统，ABA不会产生问题，但也有不能容忍ABA的系统
