原子操作类Atomic
========

相关类有：AtomicInteger,AtomicLong,AtomicBoolean,AtomicReference。

这些原子类的方法都是基于CPU的CAS原语来操作的。基于CAS的方式比使用synchronized的方式性能提高2.8倍。因此对于使用JDK5以上版本且必须支持高并发而言，应尽量使用atomic的类。
