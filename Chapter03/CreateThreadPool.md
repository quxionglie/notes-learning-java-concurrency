Executors创建线程池
=======

类库提供了一个灵活的线程池以及一些有用的默认配置，可以通过调用Executors中的静态工厂方法之一创建一个线程池。

	// 创建一个可重用固定线程数的线程池
	ExecutorService pool = Executors.newFixedThreadPool(2);


(1)	newFixedThreadPool

创建一个可重用固定线程集合的线程池，以共享的无界队列方式来运行这些线程。

(2)	newCachedThreadPool

创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。

(3)	newSingleThreadExecutor

创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。

(4)	newScheduledThreadPool

创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。

(5)	newFixedThreadPool,newCachedThreadPool返回通用的ThreadPoolExecutor实例。
