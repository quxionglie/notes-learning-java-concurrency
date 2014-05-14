下面来看看ThreadLocal的实现原理（jdk1.5源码） 
=========
```java
public class ThreadLocal<T> {  
    /** 
     * ThreadLocals rely on per-thread hash maps attached to each thread 
     * (Thread.threadLocals and inheritableThreadLocals).  The ThreadLocal 
     * objects act as keys, searched via threadLocalHashCode.  This is a 
     * custom hash code (useful only within ThreadLocalMaps) that eliminates 
     * collisions in the common case where consecutively constructed 
     * ThreadLocals are used by the same threads, while remaining well-behaved 
     * in less common cases. 
     */  
    private final int threadLocalHashCode = nextHashCode();  
  
    /** 
     * The next hash code to be given out. Accessed only by like-named method. 
     */  
    private static int nextHashCode = 0;  
  
    /** 
     * The difference between successively generated hash codes - turns 
     * implicit sequential thread-local IDs into near-optimally spread 
     * multiplicative hash values for power-of-two-sized tables. 
     */  
    private static final int HASH_INCREMENT = 0x61c88647;  
  
    /** 
     * Compute the next hash code. The static synchronization used here 
     * should not be a performance bottleneck. When ThreadLocals are 
     * generated in different threads at a fast enough rate to regularly 
     * contend on this lock, memory contention is by far a more serious 
     * problem than lock contention. 
     */  
    private static synchronized int nextHashCode() {  
        int h = nextHashCode;  
        nextHashCode = h + HASH_INCREMENT;  
        return h;  
    }  
  
    /** 
     * Creates a thread local variable. 
     */  
    public ThreadLocal() {  
    }  
  
    /** 
     * Returns the value in the current thread's copy of this thread-local 
     * variable.  Creates and initializes the copy if this is the first time 
     * the thread has called this method. 
     * 
     * @return the current thread's value of this thread-local 
     */  
    public T get() {  
        Thread t = Thread.currentThread();  
        ThreadLocalMap map = getMap(t);  
        if (map != null)  
            return (T)map.get(this);  
  
        // Maps are constructed lazily.  if the map for this thread  
        // doesn't exist, create it, with this ThreadLocal and its  
        // initial value as its only entry.  
        T value = initialValue();  
        createMap(t, value);  
        return value;  
    }  
  
    /** 
     * Sets the current thread's copy of this thread-local variable 
     * to the specified value.  Many applications will have no need for 
     * this functionality, relying solely on the {@link #initialValue} 
     * method to set the values of thread-locals. 
     * 
     * @param value the value to be stored in the current threads' copy of 
     *        this thread-local. 
     */  
    public void set(T value) {  
        Thread t = Thread.currentThread();  
        ThreadLocalMap map = getMap(t);  
        if (map != null)  
            map.set(this, value);  
        else  
            createMap(t, value);  
    }  
  
    /** 
     * Get the map associated with a ThreadLocal. Overridden in 
     * InheritableThreadLocal. 
     * 
     * @param  t the current thread 
     * @return the map 
     */  
    ThreadLocalMap getMap(Thread t) {  
        return t.threadLocals;  
    }  
  
    /** 
     * Create the map associated with a ThreadLocal. Overridden in 
     * InheritableThreadLocal. 
     * 
     * @param t the current thread 
     * @param firstValue value for the initial entry of the map 
     * @param map the map to store. 
     */  
    void createMap(Thread t, T firstValue) {  
        t.threadLocals = new ThreadLocalMap(this, firstValue);  
    }  
  
    .......  
  
    /** 
     * ThreadLocalMap is a customized hash map suitable only for 
     * maintaining thread local values. No operations are exported 
     * outside of the ThreadLocal class. The class is package private to 
     * allow declaration of fields in class Thread.  To help deal with 
     * very large and long-lived usages, the hash table entries use 
     * WeakReferences for keys. However, since reference queues are not 
     * used, stale entries are guaranteed to be removed only when 
     * the table starts running out of space. 
     */  
    static class ThreadLocalMap {  
  
    ........  
  
    }  
  
}  
```

可以看到ThreadLocal类中的变量只有这3个int型： 

	private final int threadLocalHashCode = nextHashCode();  
	private static int nextHashCode = 0;  
	private static final int HASH_INCREMENT = 0x61c88647;  

而作为ThreadLocal实例的变量只有 threadLocalHashCode 这一个，nextHashCode 和HASH_INCREMENT 是ThreadLocal类的静态变量，实际上HASH_INCREMENT是一个常量，表示了连续分配的两个ThreadLocal实例的threadLocalHashCode值的增量，而nextHashCode 的表示了即将分配的下一个ThreadLocal实例的threadLocalHashCode 的值。 

可以来看一下创建一个ThreadLocal实例即new ThreadLocal()时做了哪些操作，从上面看到构造函数ThreadLocal()里什么操作都没有，唯一的操作是这句： 

	private final int threadLocalHashCode = nextHashCode();  

那么nextHashCode()做了什么呢： 
   
```java
private static synchronized int nextHashCode() {  
    int h = nextHashCode;  
   	nextHashCode = h + HASH_INCREMENT;  
    return h;  
}  
```

就是将ThreadLocal类的下一个hashCode值即nextHashCode的值赋给实例的threadLocalHashCode，然后nextHashCode的值增加HASH_INCREMENT这个值。 

因此ThreadLocal实例的变量只有这个threadLocalHashCode，而且是final的，用来区分不同的ThreadLocal实例，ThreadLocal类主要是作为工具类来使用，那么ThreadLocal.set()进去的对象是放在哪儿的呢？ 

看一下上面的set()方法，两句合并一下成为 

	ThreadLocalMap map = Thread.currentThread().threadLocals;  

这个ThreadLocalMap 类是ThreadLocal中定义的内部类，但是它的实例却用在Thread类中： 

```java
1.	public class Thread implements Runnable {  
2.	    ......  
3.	  
4.	    /* ThreadLocal values pertaining to this thread. This map is maintained 
5.	     * by the ThreadLocal class. */  
6.	    ThreadLocal.ThreadLocalMap threadLocals = null;    
7.	    ......  
8.	}  
```

再看这句： 
```
if (map != null)  
	map.set(this, value);  
```
也就是将该ThreadLocal实例作为key，要保持的对象作为值，设置到当前线程的ThreadLocalMap 中，get()方法同样大家看了代码也就明白了，ThreadLocalMap 类的代码太多了，我就不帖了，自己去看源码吧。
