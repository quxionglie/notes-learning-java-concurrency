队列：Queue, BlockingQueue
======
JDK1.5也增加了两种新的容器类型：Queue和BlockingQueue。

Queue是用来临时保存一组等待处理的元素。Queue上的操作不会阻塞，如果队列为空，那么获取元素的操作将返回空值。

BlockingQueue扩展了Queue，增加了可阻塞的插入和获取等级操作。如果队列为空，那么获取元素的操作将一直阻塞，直到队列中出现一个可用的元素。如果队列已满，那么插入元素的操作将一直阻塞，直到队列中出现可用的空间。在生产者、消费者模式中，阻塞队列是非常有用的。

所有已知实现类： ArrayBlockingQueue, DelayQueue, LinkedBlockingDeque, LinkedBlockingQueue, PriorityBlockingQueue, SynchronousQueue


#基于阻塞队列BlockingQueue的生产者、消费者模式

```java
import java.util.concurrent.BlockingQueue;

public class Producer implements Runnable {
	private BlockingQueue queue;

	public Producer(BlockingQueue queue) {
		this.queue = queue;
	}

	public void run() {
		for (int product = 1; product <= 10; product++) {
			try {
				Thread.sleep((int) Math.random() * 3000);
				queue.put(product);
				System.out.println("Producer 生产： " + product);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}

import java.util.concurrent.BlockingQueue;

public class Consumer implements Runnable {
	private BlockingQueue queue;

	public Consumer(BlockingQueue queue) {
		this.queue = queue;
	}

	public void run() {
		for (int i = 1; i <= 10; i++) {
			try {
				Thread.sleep((int) (Math.random() * 3000));
				System.out.println("Consumer 消费： "+ queue.take()); 
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class BlockingQueueDemo {
	public static void main(String[] args) {
		BlockingQueue queue = new ArrayBlockingQueue(1);
		Thread producerThread = new Thread(
		new Producer(queue));
		Thread consumerThread = new Thread(
		new Consumer(queue));
		producerThread.start();
		consumerThread.start();
	}
}
```

```
运行结果：
Producer 生产： 1
Consumer 消费： 1
Producer 生产： 2
Consumer 消费： 2
Producer 生产： 3
Consumer 消费： 3
Producer 生产： 4
Consumer 消费： 4
Producer 生产： 5
Consumer 消费： 5
Producer 生产： 6
Consumer 消费： 6
Producer 生产： 7
Consumer 消费： 7
Producer 生产： 8
Consumer 消费： 8
Producer 生产： 9
Consumer 消费： 9
Producer 生产： 10
Consumer 消费： 10
```