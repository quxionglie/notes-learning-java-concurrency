ThreadPoolExecutor
========

#1.通用构造函数

```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

用给定的初始参数和默认的线程工厂及处理程序创建新的 ThreadPoolExecutor。使用 Executors 工厂方法之一比使用此通用构造方法方便得多。

参数：

corePoolSize - 池中所保存的线程数，包括空闲线程。 

maximumPoolSize - 池中允许的最大线程数。 

keepAliveTime - 当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间。 

unit - keepAliveTime 参数的时间单位。 

workQueue - 执行前用于保持任务的队列。此队列仅保持由 execute 方法提交的 Runnable 任务。 

threadFactory - 执行程序创建新线程时使用的工厂。 

handler - 由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序。 

###抛出：

IllegalArgumentException - 如果 corePoolSize 或 keepAliveTime 小于零，或者 maximumPoolSize 小于或等于零，或者 corePoolSize 大于 maximumPoolSize。 

NullPointerException - 如果 workQueue、threadFactory 或 handler 为 null。

#2.	线程的创建和销毁
线程池的基本大小（corePoolSize）、最大大小（maximumPoolSize）以及存活时间等因素共同负责线程的创建与销毁。

基本大小也是线程池的目标大小，即在没有任务执行时线程池的大小，并且只有在工作队列满了的情况下才会创建超出这个数量的线程。

最大大小表示可同时活动的线程数量的上限。

如果某个线程的空闲时间超过了存活时间，那么将被标记为可回收的，并且当线程池的当前大小超过基本大小时，这个线程将被终止。

#3.	管理队列任务

ThreadPoolExecutor允许提供一个BlockingQueue来保存等待执行的任务。基本的任务排队方法有3种：有界队列, 无界队列, 同步移交（Synchronous Handoff）。

#4.	有界队列饱和策略

有界队列被填满后，饱和策略开始发挥作用。饱和策略可以通过调用setRejectedExecutionHandler来修改。jdk提供了几种不同的RejectedExecutionHandler实现：AbortPolicy,  CallerRunsPolicy,  DiscardOldestPolicy,  DiscardPolicy 。

AbortPolicy：默认的饱和策略。抛出 RejectedExecutionException。调用者可以捕获这个异常，然后根据需求编写自己的处理代码。

CallerRunsPolicy: 不抛弃任务，不抛出异常，而将任务退回给调用者。

DiscardOldestPolicy：放弃最旧的未处理请求，然后重试 execute；如果执行程序已关闭，则会丢弃该任务。

DiscardPolicy:默认情况下它将放弃被拒绝的任务。

#5.	扩展ThreadPoolExecutor

ThreadPoolExecutor是可扩展的，它提供了几个可以在子类化中改写的方法：beforeExecute、afterExcute和terminated。在这些方法中，还可以添加日志、计时、监视或统计信息收集的功能。

无论任务是从run中正常返回，还是会抛出一个异常在而返回，afterExcute都会被调用。（如果任务执行完成后带有Error，那么就不会调用afterExcute）。

如果beforeExecute抛出一个RuntimeException，那么任务将不被执行，并且afterExcute也会被调用。

在线程池完成关闭操作时调用terminated，也就是在所有任务都已经完成并且所有工作者线程也已经关闭后，terminated可以释放Executor在其生命周期里分配的各种资源，此外还可以执行发送通知，记录日志或者收集finalize统计信息等操作。

```java
//增加了日志和计时等功能的线程池
public class TimingThreadPool extends ThreadPoolExecutor {
	
	private final ThreadLocal<Long> startTime = new ThreadLocal<Long>();
	private final Logger log = Logger.getLogger("TimingThreadPool");
	private final AtomicLong numTasks = new AtomicLong();
	private final AtomicLong totalTime = new AtomicLong();
	//构造函数
	//...

	protected void beforeExecute(Thread t, Runnable r) {
		super.beforeExecute(t, r);
		log.fine(String.format("Thread %s: start %s", t, r));
		startTime.set(System.nanoTime());
	}

	protected void afterExecute(Runnable r, Throwable t) {
		try {
			long endTime = System.nanoTime();
			long taskTime = endTime - startTime.get();
			numTasks.incrementAndGet();
			totalTime.addAndGet(taskTime);
			log.fine(String.format("Thread %s: end %s, time=%dns", t, r,
					taskTime));
		} finally {
			super.afterExecute(r, t);
		}
	}

	protected void terminated() {
		try {
			log.info(String.format("Terminated: avg time=%dns", totalTime.get()
					/ numTasks.get()));
		} finally {
			super.terminated();
		}
	}
}
```
