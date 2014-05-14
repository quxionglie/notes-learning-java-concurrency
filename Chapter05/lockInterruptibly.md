可中断的锁
========

```java
public boolean sendOnSharedLine(Stringmessage)  
      throws InterruptedException{ 
    lock.lockInterruptibly(); //如果当前线程未被中断，则获取锁定。  
    try{ 
        return cancellableSendOnSharedLine(message); 
    }finally{ 
        lock.unlock(); 
    } 
} 
private boolean cancellableSendOnSharedLine(String message) 
    throwsInterruptedException{...} 
} 
```
