任务取消(todo 中断策略 ...)
===========

如果外部代码能在某个操作正常完成之前将其置入"完成"状态，那么这个操作就可以称为可取消的(cancellable)。

###取消的原因有多种：

(1). 用户请求取消。如通过管理接口来发送取消请求。

(2). 	有时间限制的操作。例如某个程序需要在有限时间内搜索问题空间，并在这个时间内选择最佳的解决方案。当计时器超时时，需要取消所有正在搜索的任务。

(3). 应用程序事件。例如某个程序对某个问题空间进行分解并搜索，从而使不同的任务可以搜索问题空间中的不同区域。当其中一个任务找到了解决方案时，所有其它仍在搜索的任务都将被取消。

(4). 错误。网页爬虫程序搜索相关的页面，并将页面或摘要数据保存到硬盘。当一个爬虫任务发生错误时(例如磁盘满)，那么所有的搜索任务都会取消，此时可能会记录它们的当前状态，以便稍后重新启动。

(5). 关闭。当一个程序或服务关闭时，必须对正在处理和等待处理的工作执行来某种操作。在平缓的关闭中，当前正在执行的任务将继续执行直到完成，而在立即关闭过程中，当前的任务可能取消。

#1.	使用volatile类型的域来保存取消状态

```java
public class PrimeGenerator implements Runnable {
	private final List<BigInteger> primes = new ArrayList<BigInteger>();
	private volatile boolean cancelled;

	public void run() {
		BigInteger p = BigInteger.ONE;
		while (!cancelled) {
			p = p.nextProbablePrime();
			synchronized (this) {
				primes.add(p);
			}
		}
	}

	public void cancel() {
		cancelled = true;
	}

	public synchronized List<BigInteger> get() {
		return new ArrayList<BigInteger>(primes);
	}
}
```

#2.	通过中断来取消
Thread中的中断方法

```java
public class Thread { 
    public void interrupt() { ... } 
    public boolean isInterrupted() { ... } 
    public static boolean interrupted() { ... } 
    ... 
} 
 
class PrimeProducer extends Thread { 
    private final BlockingQueue<BigInteger> queue; 
    PrimeProducer(BlockingQueue<BigInteger> queue) { 
        this.queue = queue; 
    } 
    public void run() { 
        try { 
            BigInteger p = BigInteger.ONE; 
            while (!Thread.currentThread().isInterrupted()) 
                queue.put(p = p.nextProbablePrime()); 
        } catch (InterruptedException consumed) { 
            /* Allow thread to exit */ 
        } 
    } 
    public void cancel() { 
        interrupt(); 
    } 
} 
```

#3.	通过Future来取消

```java
public static void timedRun(Runnable r, 
                            long timeout, TimeUnit unit) 
throws InterruptedException { 
    Future<?> task = taskExec.submit(r); 
    try { 
        task.get(timeout, unit); 
    } catch (TimeoutException e) { 
        // 接下来任务将被取消 
    } catch (ExecutionException e) { 
        // 如果在任务中抛出了异常，那么重新抛出异常 
        throw launderThrowable(e.getCause()); 
    } 
    finally { 
        // 如果任务已经结束，那么取消也不会带来任何影响 
        task.cancel(true); // 如果任务正在运行，那么将被终短
    } 
} 
```

