任务执行的三种模式
============

#(1)串行的执行任务

```java
//串行的web服务器
class SingleThreadWebServer {
	public static void main(String[] args) throws IOException {
		ServerSocket socket = new ServerSocket(80);
		while (true) {
			Socket connection = socket.accept();
			handleRequest(connection);
		}
	}
}
```
SingleThreadWebServer很简单，且在理论上是正确的，但在实际生产环境中的执行性能却很糟糕，因为它每次只能处理一次请求。在服务器应用程序中，串行处理机制都无法提供高吞吐率或快速响应性。

#(2)显式的为任务创建线程

```java
//在web服务器中为每个请求启动一个新的线程(不要这么做)
class ThreadPerTaskWebServer {
	public static void main(String[] args) throws IOException {
		ServerSocket socket = new ServerSocket(80);
		while (true) {
			final Socket connection = socket.accept();
			Runnable task = new Runnable() {
				public void run() {
					handleRequest(connection);
				}
			};
			new Thread(task).start();
		}
	}
}
```

在正常的情况下，”为每个任务分配一个线程”的方法能提升串行执行任务的性能。只要请求的到达速率不超过服务器的请求处理能力，那么这种方法可以同时带来更快的响应性和更高的吞吐率。

但这种方法存在一些缺陷，尤其当需要创建大量线程的时候。
* 线程生命周期的开销非常高。
* 资源消耗。活跃的线程会消耗系统资源，尤其是内存。如果可运行的线程数多于可用处理器的数量，那么有些线程将闲置。大量空闲的线程会占用许多内存，给垃圾回收器带来压力，而且大量线程在竞争CPU资源时还将产生其他的性能开销。
* 稳定性。在可创建线程的数量上存在一个限制。如果超过了这些限制，很可能会抛出outOfMemoryError异常，要想从这种错误中恢复过来是非常危险的，更简单的方法是通过构造程序来避免超出这些限制。


#(3).基于线程池的实现

```java
// 基于线程池的web服务器
class TaskExecutionWebServer {
	private static final int NTHREADS = 100;
	private static final Executor exec = Executors.newFixedThreadPool(NTHREADS);

	public static void main(String[] args) throws IOException {
		ServerSocket socket = new ServerSocket(80);
		while (true) {
			final Socket connection = socket.accept();
			Runnable task = new Runnable() {
				public void run() {
					handleRequest(connection);
				}
			};
			exec.execute(task);
		}
	}
}
```

在TaskExecutionWebServer中，通过使用Executor，将请求处理任务的提交与任务的实际执行解耦开来，并且只需采用另一种不同的Executor实现，就可以改变服务器的行为。
