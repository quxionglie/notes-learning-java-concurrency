示例
======

Java线程：新特征-原子量

http://lavasoft.blog.51cto.com/62575/222541

反面例子（切勿模仿）：

```java
/**
 * Java线程：新特征-原子量
*/
public class Test {
    public static void main(String[] args) {
		ExecutorService pool = Executors.newFixedThreadPool(2);
		Runnable t1 = new MyRunnable("张三", 2000);
		Runnable t2 = new MyRunnable("李四", 3600);
		Runnable t3 = new MyRunnable("王五", 2700);
		Runnable t4 = new MyRunnable("老张", 600);
		Runnable t5 = new MyRunnable("老牛", 1300);
		Runnable t6 = new MyRunnable("胖子", 800);
		// 执行各个线程
		pool.execute(t1);
		pool.execute(t2);
		pool.execute(t3);
		pool.execute(t4);
		pool.execute(t5);
		pool.execute(t6);
		// 关闭线程池
		pool.shutdown();
    }
}

class MyRunnable implements Runnable {
    // 原子量，每个线程都可以自由操作
    private static AtomicLong aLong = new AtomicLong(10000);
    private String name; // 操作人
    private int x; // 操作数额

    MyRunnable(String name, int x) {
		this.name = name;
		this.x = x;
    }

    public void run() {
		System.out.println(name + "执行了" + x + "，当前余额：" + aLong.addAndGet(x));
    }
}
```

```
运行结果：
李四执行了3600，当前余额：13600 
王五执行了2700，当前余额：16300 
老张执行了600，当前余额：16900 
老牛执行了1300，当前余额：18200 
胖子执行了800，当前余额：19000 
张三执行了2000，当前余额：21000 

张三执行了2000，当前余额：12000 
王五执行了2700，当前余额：18300 
老张执行了600，当前余额：18900 
老牛执行了1300，当前余额：20200 
胖子执行了800，当前余额：21000 
李四执行了3600，当前余额：15600 

张三执行了2000，当前余额：12000 
李四执行了3600，当前余额：15600 
老张执行了600，当前余额：18900 
老牛执行了1300，当前余额：20200 
胖子执行了800，当前余额：21000 
王五执行了2700，当前余额：18300 
```

从运行结果可以看出，虽然使用了原子量，但是程序并发访问还是有问题，那究竟问题出在哪里了？
 
这里要注意的一点是，原子量虽然可以保证单个变量在某一个操作过程的安全，但无法保证你整个代码块，或者整个程序的安全性。因此，通常还应该使用锁等同步机制来控制整个程序的安全性。

下面是对这个错误修正：

```java
/**
 * Java线程：新特征-原子量
 */
public class Test {
    public static void main(String[] args) {
		ExecutorService pool = Executors.newFixedThreadPool(2);
		Lock lock = new ReentrantLock(false);
		Runnable t1 = new MyRunnable("张三", 2000, lock);
		Runnable t2 = new MyRunnable("李四", 3600, lock);
		Runnable t3 = new MyRunnable("王五", 2700, lock);
		Runnable t4 = new MyRunnable("老张", 600, lock);
		Runnable t5 = new MyRunnable("老牛", 1300, lock);
		Runnable t6 = new MyRunnable("胖子", 800, lock);
		// 执行各个线程
		pool.execute(t1);
		pool.execute(t2);
		pool.execute(t3);
		pool.execute(t4);
		pool.execute(t5);
		pool.execute(t6);
		// 关闭线程池
		pool.shutdown();
    }
}

class MyRunnable implements Runnable {
    // 原子量，每个线程都可以自由操作
    private static AtomicLong aLong = new AtomicLong(10000);
    private String name; // 操作人
    private int x; // 操作数额
    private Lock lock;

    MyRunnable(String name, int x, Lock lock) {
		this.name = name;
		this.x = x;
		this.lock = lock;
    }

    public void run() {
		lock.lock();
		System.out.println(name + "执行了" + x + "，当前余额：" + aLong.addAndGet(x));
		lock.unlock();
    }
}

```

```
执行结果：
张三执行了2000，当前余额：12000 
王五执行了2700，当前余额：14700 
老张执行了600，当前余额：15300 
老牛执行了1300，当前余额：16600 
胖子执行了800，当前余额：17400 
李四执行了3600，当前余额：21000 
```

这里使用了一个对象锁，来控制对并发代码的访问。不管运行多少次，执行次序如何，最终余额均为21000，这个结果是正确的。
