读-写锁
======

ReadWriteLock 维护了一对相关的锁定，一个用于只读操作，另一个用于写入操作。

```java
//ReadWriteLock 接口 
public interface ReadWriteLock { 
    Lock readLock(); 
    Lock writeLock(); 
} 
```


示例：用读-写锁来包装Map

```java
public class ReadWriteMap<K, V> { 
    private final Map<K, V> map; 
    private final ReadWriteLock lock = new ReentrantReadWriteLock(); 
    private final Lock r = lock.readLock(); 
    private final Lock w = lock.writeLock(); 
    public ReadWriteMap(Map<K, V> map) { 
        this.map = map; 
    } 
    public V put(K key, V value) { 
        w.lock(); 
        try { 
            return map.put(key, value); 
        } 
        finally { 
            w.unlock(); 
        } 
    } 
    // 对 remove(), putAll(), clear()等方法执行相同的操作 
 
    public V get(Object key) { 
        r.lock(); 
        try { 
            return map.get(key); 
        } 
        finally { 
            r.unlock(); 
        } 
    } 
    // 对其它只读的方法执行相同的操作 
} 
```
