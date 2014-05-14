线程池创建示例
==========
#1.	固定大小的线程池

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ExecutorService;

/**
 * Java线程：线程池-固定线程数的线程池
 */
public class Test {
	public static void main(String[] args) {
		// 创建一个可重用固定线程数的线程池
		ExecutorService pool = Executors.newFixedThreadPool(2);
		// 创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口
		Thread t1 = new MyThread();
		Thread t2 = new MyThread();
		Thread t3 = new MyThread();
		Thread t4 = new MyThread();
		Thread t5 = new MyThread();
		// 将线程放入池中进行执行
		pool.execute(t1);
		pool.execute(t2);
		pool.execute(t3);
		pool.execute(t4);
		pool.execute(t5);
		// 关闭线程池
		pool.shutdown();
	}
}

class MyThread extends Thread {
	@Override
	public void run() {
		System.out.println(Thread.currentThread().getName() + "正在执行。。。");
	}
}

```

```
pool-1-thread-1正在执行。。。
pool-1-thread-2正在执行。。。
pool-1-thread-1正在执行。。。
pool-1-thread-2正在执行。。。
pool-1-thread-1正在执行。。。
```

#2.	单任务线程池

在上例的基础上改一行创建pool对象的代码为：
```
//创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。 
ExecutorService pool = Executors.newSingleThreadExecutor();

输出结果为：
pool-1-thread-1正在执行。。。 
pool-1-thread-1正在执行。。。 
pool-1-thread-1正在执行。。。 
pool-1-thread-1正在执行。。。 
pool-1-thread-1正在执行。。。
```

#3.	可变尺寸的线程池

与上面的类似，只是改动下pool的创建方式：
```
//创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。 
ExecutorService pool = Executors.newCachedThreadPool();

pool-1-thread-2正在执行。。。
pool-1-thread-1正在执行。。。
pool-1-thread-3正在执行。。。
pool-1-thread-4正在执行。。。
pool-1-thread-5正在执行。。。
```

#4.	延迟连接池

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Java线程：线程池
 */
public class Test {
	public static void main(String[] args) {
        //创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。 
        ScheduledExecutorService pool = Executors.newScheduledThreadPool(2); 
        //创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口 
        Thread t1 = new MyThread(); 
        Thread t2 = new MyThread(); 
        Thread t3 = new MyThread(); 
        Thread t4 = new MyThread(); 
        Thread t5 = new MyThread(); 
        //将线程放入池中进行执行 
        pool.execute(t1); 
        pool.execute(t2); 
        pool.execute(t3); 
        //使用延迟执行风格的方法 
        pool.schedule(t4, 10, TimeUnit.MILLISECONDS); 
        pool.schedule(t5, 10, TimeUnit.MILLISECONDS); 
        //关闭线程池 
        pool.shutdown(); 
	}
}

class MyThread extends Thread {
	@Override
	public void run() {
		System.out.println(Thread.currentThread().getName() + "正在执行。。。");
	}
}

```

```
pool-1-thread-2正在执行。。。
pool-1-thread-1正在执行。。。
pool-1-thread-2正在执行。。。
pool-1-thread-1正在执行。。。
pool-1-thread-2正在执行。。。
```

#5.	单任务延迟连接池

在四代码基础上，做改动

```
//创建一个单线程执行程序，它可安排在给定延迟后运行命令或者定期地执行。 
ScheduledExecutorService pool = Executors.newSingleThreadScheduledExecutor();
 
pool-1-thread-1正在执行。。。 
pool-1-thread-1正在执行。。。 
pool-1-thread-1正在执行。。。 
pool-1-thread-1正在执行。。。 
pool-1-thread-1正在执行。。。
```

#6.	自定义线程池

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * Java线程：线程池-自定义线程池
 */
public class Test {
	public static void main(String[] args) {
		// 创建等待队列
		BlockingQueue<Runnable> bqueue = new ArrayBlockingQueue<Runnable>(20);
		// 创建一个单线程执行程序，它可安排在给定延迟后运行命令或者定期地执行。
		ThreadPoolExecutor pool = new ThreadPoolExecutor(2, 3, 2, TimeUnit.MILLISECONDS, bqueue);
		// 创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口
		Thread t1 = new MyThread();
		Thread t2 = new MyThread();
		Thread t3 = new MyThread();
		Thread t4 = new MyThread();
		Thread t5 = new MyThread();
		Thread t6 = new MyThread();
		Thread t7 = new MyThread();
		// 将线程放入池中进行执行
		pool.execute(t1);
		pool.execute(t2);
		pool.execute(t3);
		pool.execute(t4);
		pool.execute(t5);
		pool.execute(t6);
		pool.execute(t7);
		// 关闭线程池
		pool.shutdown();
	}
}

class MyThread extends Thread {
	@Override
	public void run() {
		System.out.println(Thread.currentThread().getName() + "正在执行。。。");
		try {
			Thread.sleep(100L);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

```
pool-1-thread-1正在执行。。。
pool-1-thread-2正在执行。。。
pool-1-thread-1正在执行。。。
pool-1-thread-2正在执行。。。
pool-1-thread-1正在执行。。。
pool-1-thread-2正在执行。。。
pool-1-thread-2正在执行。。。
```
