栅栏CyclicBarrier与计数器CountDownLatch
=========

#1.	栅栏 CyclicBarrier

栅栏类似于闭锁，它能阻塞一组线程直到某个事件发生。所有线程必须同时到达栅栏位置，才能继续执行。栅栏用于实现一些协议，例如几个家庭成员决定在某个地方集合：”所有人6:00在麦当劳集合，到了以后要等其他人，之后再讨论下一步要做的事”。

CyclicBarrier可以使一定数量的参与方反复地在栅栏位置汇集，它在并行迭代算法中非常有用。

在模拟程序中通常需要栅栏，例如某个步骤中的计算可以并行执行，但必须等到该步骤中的所有计算都执行完毕才能进入下一个步骤。

```java
ExecutorService service = Executors.newCachedThreadPool();
// 构造方法里的数字标识有几个线程到达集合地点开始进行下一步工作
final CyclicBarrier cb = new CyclicBarrier(3);
for (int i = 0; i < 3; i++) {
	Runnable runnable = new Runnable() {
		public void run() {
			try {
				Thread.sleep((long) (Math.random() * 10000));
				System.out.println("线程"
						+ Thread.currentThread().getName()
						+ "即将到达集合地点1，当前已有" + cb.getNumberWaiting()
						+ "个已经到达，正在等候");
				cb.await();

				Thread.sleep((long) (Math.random() * 10000));
				System.out.println("线程"
						+ Thread.currentThread().getName()
						+ "即将到达集合地点2，当前已有" + cb.getNumberWaiting()
						+ "个已经到达，正在等候");
				cb.await();
				Thread.sleep((long) (Math.random() * 10000));
				System.out.println("线程"
						+ Thread.currentThread().getName()
						+ "即将到达集合地点3，当前已有" + cb.getNumberWaiting()
						+ "个已经到达，正在等候");
				cb.await();
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	};
	service.execute(runnable);

}
service.shutdown();
```

```
执行结果：
线程pool-1-thread-2即将到达集合地点1，当前已有0个已经到达，正在等候
线程pool-1-thread-1即将到达集合地点1，当前已有1个已经到达，正在等候
线程pool-1-thread-3即将到达集合地点1，当前已有2个已经到达，正在等候
线程pool-1-thread-1即将到达集合地点2，当前已有0个已经到达，正在等候
线程pool-1-thread-3即将到达集合地点2，当前已有1个已经到达，正在等候
线程pool-1-thread-2即将到达集合地点2，当前已有2个已经到达，正在等候
线程pool-1-thread-1即将到达集合地点3，当前已有0个已经到达，正在等候
线程pool-1-thread-2即将到达集合地点3，当前已有1个已经到达，正在等候
线程pool-1-thread-3即将到达集合地点3，当前已有2个已经到达，正在等候
```

#2.	计数器 CountDownLatch

```java
ExecutorService service = Executors.newCachedThreadPool();
final CountDownLatch cdOrder = new CountDownLatch(1);
final CountDownLatch cdAnswer = new CountDownLatch(3);
for (int i = 0; i < 3; i++) {
	Runnable runnable = new Runnable() {
		public void run() {
			try {
				System.out.println("线程"
						+ Thread.currentThread().getName() + "正准备接受命令");
				cdOrder.await();
				System.out.println("线程"
						+ Thread.currentThread().getName() + "已接受命令");
				Thread.sleep((long) (Math.random() * 10000));
				System.out
						.println("线程"
								+ Thread.currentThread().getName()
								+ "回应命令处理结果");
				cdAnswer.countDown();
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	};
	service.execute(runnable);
}
try {
	Thread.sleep((long) (Math.random() * 10000));

	System.out.println("线程" + Thread.currentThread().getName()
			+ "即将发布命令");
	cdOrder.countDown();
	System.out.println("线程" + Thread.currentThread().getName()
			+ "已发送命令，正在等待结果");
	cdAnswer.await();
	System.out.println("线程" + Thread.currentThread().getName()
			+ "已收到所有响应结果");
} catch (Exception e) {
	e.printStackTrace();
}
service.shutdown();
```

```
结果：
线程pool-1-thread-1正准备接受命令
线程pool-1-thread-2正准备接受命令
线程pool-1-thread-3正准备接受命令
线程main即将发布命令
线程main已发送命令，正在等待结果
线程pool-1-thread-1已接受命令
线程pool-1-thread-2已接受命令
线程pool-1-thread-3已接受命令
线程pool-1-thread-1回应命令处理结果
线程pool-1-thread-2回应命令处理结果
线程pool-1-thread-3回应命令处理结果
线程main已收到所有响应结果
```
