Atomic典型应用:按顺序获取ID
=======

传统方式必须在每次获取时加锁，以防止出现并发时取到同样id的现象。

用AtomicInteger实现示例如下：

```java
private static AtomicInteger counter = new AtomicInteger();
public static int getNextId() {
	return counter.incrementAndGet();
}
```