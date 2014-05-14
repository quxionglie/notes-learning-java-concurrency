携带结果的任务Callable与Future
=========

Executor框架使用Runnable作为其任务的基本表达形式。Runnable是一个有很大局限的抽象，它不能返回一个值或抛出一个受检查的异常。

Callable是更好的抽象：它认为主进入点(即call)将返回一个值，并可能抛出一个异常。

Future表示一个任务的生命周期，并提供了相应的方法来判断是否已经完成或取消，以及获取任务的结果和取消任务等。在Future规范中包含的隐含意义是，任务的生命周期只能前进，不能后退，当某个任务完成后，它将永远停留在“完成”的状态上。

get方法的行为取决于任务的状态（尚未开始、正在运行、已完成）。

如果任务已经完成，那么get方法会立即返回或都抛出一个Exception。

如果任务没有完成，那么get将阻塞并直到任务完成。

如果任务抛出了异常，那么get将该异常封装为ExecutionException并重新抛出。如果get抛出ExecutionException，那么可以通过getCause获得被封装的初始异常。

如果任务被取消，那么get将抛出CanclelationException。

java code：callable与Future接口

```java
public interface Callable<V> { 
    V call() throws Exception; 
} 
public interface Future<V> { 
    boolean cancel(boolean mayInterruptIfRunning); 
    boolean isCancelled(); 
    boolean isDone(); 
    V get() throws InterruptedException, ExecutionException, 
    CancellationException; 
    V get(long timeout, TimeUnit unit) 
    throws InterruptedException, ExecutionException, 
    CancellationException, TimeoutException; 
}
```

#实例

可返回值的任务必须实现Callable接口，类似的，无返回值的任务必须Runnable接口。

执行Callable任务后，可以获取一个Future的对象，在该对象上调用get就可以获取到Callable任务返回的Object了。

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

/**
 * Java线程：有返回值的线程
 */
public class Test {
	public static void main(String[] args) throws ExecutionException,
			InterruptedException {
		// 创建一个线程池
		ExecutorService pool = Executors.newFixedThreadPool(2);
		// 创建两个有返回值的任务
		Callable c1 = new MyCallable("A");
		Callable c2 = new MyCallable("B");
		// 执行任务并获取Future对象
		Future f1 = pool.submit(c1);
		Future f2 = pool.submit(c2);
		// 从Future对象上获取任务的返回值，并输出到控制台
		System.out.println(">>>" + f1.get().toString());
		System.out.println(">>>" + f2.get().toString());
		// 关闭线程池
		pool.shutdown();
	}
}

class MyCallable implements Callable {
	private String oid;

	MyCallable(String oid) {
		this.oid = oid;
	}

	public Object call() throws Exception {
		return oid + "任务返回的内容";
	}
}

```

```
>>>A任务返回的内容
>>>B任务返回的内容
```
