Lock与ReentrantLock
=========

Lock接口中定义了一组抽象的加锁机制。Lock 实现提供了比使用 synchronized 方法和语句可获得的更广泛的锁定操作。

ReentrantLock实现了Lock接口，并提供了与synchronized相同的互斥性和内存可见性。

```java
public interface Lock { 
void lock(); 
//如果当前线程未被中断，则获取锁定。 
    void lockInterruptibly() throws InterruptedException; 
    boolean tryLock(); 
    boolean tryLock(long timeout, TimeUnit unit) 
    throws InterruptedException; 
    void unlock(); 
    Condition newCondition(); 
} 
```

使用ReentrantLock来保护对象状态。
```java
Lock lock = new ReentrantLock(); 
lock.lock(); 
try { 
    //相关操作
} finally { 
    lock.unlock();  //一定要释放锁
} 
```
