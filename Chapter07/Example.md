例子
======

```java
public class SeqGen {
	private static ThreadLocal<Integer> seqNum = new ThreadLocal<Integer>() {
		@Override
		public Integer initialValue() {
			return 0;
		}
	};

	public int getNextNum() {
		seqNum.set(seqNum.get() + 1);
		return seqNum.get();
	}
}

public class TestThread implements Runnable {
	private SeqGen sn;

	public TestThread(SeqGen sn) {
		this.sn = sn;
	}

	public void run() {
		for (int i = 0; i < 3; i++) {
			String str = Thread.currentThread().getName() + "-->"
					+ sn.getNextNum();
			System.out.println(str);
		}
	}
}

public class Test {
	public static void main(String[] args) {
		SeqGen sn = new SeqGen();
		TestThread tt1 = new TestThread(sn);
		TestThread tt2 = new TestThread(sn);
		TestThread tt3 = new TestThread(sn);
		Thread t1 = new Thread(tt1);
		Thread t2 = new Thread(tt2);
		Thread t3 = new Thread(tt3);
		t1.start();
		t2.start();
		t3.start();
	}
}
```

```
输出结果：
Thread-2-->1
Thread-2-->2
Thread-2-->3
Thread-1-->1
Thread-0-->1
Thread-0-->2
Thread-0-->3
Thread-1-->2
Thread-1-->3
```

资料来源：

(1)	理解ThreadLocal
http://blog.csdn.net/qjyong/article/details/2158097

(2)	ThreadLocal的几种误区
http://www.blogjava.net/jspark/archive/2006/08/01/61165.html

(3)	正确理解ThreadLocal
http://lujh99.iteye.com/blog/103804
