集合与线程安全
=============

线程安全的集合包的三种实现方式。

	摘抄：《Java 理论与实践: 并发集合类》

	在Java类库中出现的第一个关联的集合类是 Hashtable ，它是JDK 1.0的一部分。 Hashtable 提供了一种易于使用的、线程安全的、关联的map功能，这当然也是方便的。然而，线程安全性是凭代价换来的―― Hashtable 的所有方法都是同步的。 此时，无竞争的同步会导致可观的性能代价。 

	Hashtable 的后继者 HashMap 是作为JDK1.2中的集合框架的一部分出现的，它通过提供一个不同步的基类和一个同步的包装器 Collections.synchronizedMap ，解决了线程安全性问题。 通过将基本的功能从线程安全性中分离开来， Collections.synchronizedMap 允许需要同步的用户可以拥有同步，而不需要同步的用户则不必为同步付出代价。

	Hashtable 和 synchronizedMap 所采取的获得同步的简单方法（同步 Hashtable 中或者同步的 Map 包装器对象中的每个方法）有两个主要的不足。

	首先，这种方法对于可伸缩性是一种障碍，因为一次只能有一个线程可以访问hash表。
	
	同时，这样仍不足以提供真正的线程安全性，许多公用的混合操作仍然需要额外的同步。虽然诸如 get() 和 put() 之类的简单操作可以在不需要额外同步的情况下安全地完成，但还是有一些公用的操作序列 ，例如迭代或者put-if-absent（空则放入），需要外部的同步，以避免数据争用。


#(1)	线程安全类 Vector,HashTable
Vector，因为效率较低，现在已经不太建议使用。

#(2)	使用ArrayList/HashMap和同步包装器  

	List synchArrayList = Collections.synchronizedList(new ArrayList());  
	Map synchHashMap = Collections.synchronizedMap(new HashMap())  ;

如果要迭代，需要这样

	synchronized (synchHashMap)  {  
	   Iterator iter = synchHashMap.keySet().iterator();  
	   while (iter.hasNext()) . . .;  
	} 

#(3)	java.util.concurrent包
用java5.0新加入的ConcurrentHashMap、ConcurrentLinkedQueue、CopyOnWriteArray- List 和 CopyOnWriteArraySet ，对这些集合进行并发修改是安全的。

ConcurrentHashMap是线程安全的HashMap实现。无论元素数量为多少，在线程数为10时, ConcurrentHashMap带来的性能提升并不是很明显。 但在线程数为50和100时，Concurrent- HashMap在添加和删除元素时带来了一倍左右的性能提升，在查找元素上更是带来了10倍左右的性能提升，并且随着线程数增长，ConcurrentHashMap性能并没有出现下降的现象。

CopyOnWriteArrayList是一个线程安全、并且在读操作时无锁的ArrayList。：在读多写少的并发环境中，一般用 CopyOnWriteArrayList 类替代 ArrayList 。
