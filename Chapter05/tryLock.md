
轮询锁和定时锁
=========

可定时的与可轮询的锁获取模式是由tryLock方法实现的，与无条件的锁获取模式相比，它具有更完善的错误恢复机制。

#1.	轮询锁

```java
//示例：通过tryLock来避免锁顺序死锁。 
//利用tryLock来获取两个锁，如果不能同时获得，那么回退并重新尝试。如果在指定的时间内不能获得所需要的锁，那么tansferMoney将返回一个失败状态，从而使该操作平缓地失败。 
 public boolean transferMoney(Account fromAcct, 
                             Account toAcct, 
                             DollarAmount amount, 
                             long timeout, 
                             TimeUnit unit) 
throws InsufficientFundsException, InterruptedException { 
    long fixedDelay = getFixedDelayComponentNanos(timeout, unit); 
    long randMod = getRandomDelayModulusNanos(timeout, unit); 
    long stopTime = System.nanoTime() + unit.toNanos(timeout); 
    while (true) { 
        if (fromAcct.lock.tryLock()) { 
            try { 
                if (toAcct.lock.tryLock()) { 
                    try { 
                        if (fromAcct.getBalance().compareTo(amount) < 0) 
                            throw new InsufficientFundsException(); 
                        else { 
                            fromAcct.debit(amount); 
                            toAcct.credit(amount); 
                            return true; 
                        } 
                    } 
                    finally { 
                        toAcct.lock.unlock(); 
                    } 
                } 
            } 
            finally { 
                fromAcct.lock.unlock(); 
            } 
        } 
        if (System.nanoTime() < stopTime) 
            return false; 
        NANOSECONDS.sleep(fixedDelay + rnd.nextLong() % randMod); 
    } 
} 

```

#2.	定时锁

Java 5提供了更灵活的锁工具，可以显式地索取和释放锁。那么在索取锁的时候可以设定一个超时时间，如果超过这个时间还没索取到锁，则不会继续堵塞而是放弃此次任务，示例代码如下：

```java
public boolean trySendOnSharedLine(String message, 
                                   long timeout, TimeUnit unit) 
throws InterruptedException { 
long nanosToLock = unit.toNanos(timeout) 
- estimatedNanosToSend(message); 
    if (!lock.tryLock(nanosToLock, NANOSECONDS)) 
        return false; 
    try { 
        return sendOnSharedLine(message); 
    } finally { 
        lock.unlock(); 
    } 
} 
```
