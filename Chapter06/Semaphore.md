信号量Semaphore
========
计数信号量（Counting Semaphore）用来控制同时访问某个特定资源的操作数量，或者同时执行某个指定操作的数量。计数信号量还可以用来实现某种资源池，或者对容器施加边界。

Semaphore中管理着一组虚拟的许可（permit），许可的初始数量可通过构造函数来指定。在执行操作时可以首先获得许可（只要还有剩余的许可），并在使用以后释放许可。如果没有许可，那么acquire将阻塞直到许可（或者被中断或者超时）。release方法将返回一个许可信号量。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaphoreTest {
	public static void main(String[] args) {
		MyPool myPool = new MyPool(20);
		// 创建线程池
		ExecutorService threadPool = Executors.newFixedThreadPool(2);
		MyThread t1 = new MyThread("任务A", myPool, 3);
		MyThread t2 = new MyThread("任务B", myPool, 12);
		MyThread t3 = new MyThread("任务C", myPool, 7);
		// 在线程池中执行任务
		threadPool.execute(t1);
		threadPool.execute(t2);
		threadPool.execute(t3);
		// 关闭池
		threadPool.shutdown();
	}
}

/**
 * 一个池
 */
class MyPool {
	private Semaphore sp; // 池相关的信号量

	/**
	 * 池的大小，这个大小会传递给信号量
	 * 
	 * @param size
	 *            池的大小
	 */
	MyPool(int size) {
		this.sp = new Semaphore(size);
	}

	public Semaphore getSp() {
		return sp;
	}

	public void setSp(Semaphore sp) {
		this.sp = sp;
	}
}

class MyThread extends Thread {
	private String threadname; // 线程的名称
	private MyPool pool; // 自定义池
	private int x; // 申请信号量的大小

	MyThread(String threadname, MyPool pool, int x) {
		this.threadname = threadname;
		this.pool = pool;
		this.x = x;
	}

	public void run() {
		try {
			// 从此信号量获取给定数目的许可
			pool.getSp().acquire(x);
			// todo：也许这里可以做更复杂的业务
			System.out.println(threadname + "成功获取了" + x + "个许可！");
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			// 释放给定数目的许可，将其返回到信号量。
			pool.getSp().release(x);
			System.out.println(threadname + "释放了" + x + "个许可！");
		}
	}
}
```

```
结果：
任务A成功获取了3个许可！
任务B成功获取了12个许可！
任务A释放了3个许可！
任务B释放了12个许可！
任务C成功获取了7个许可！
任务C释放了7个许可！
```
