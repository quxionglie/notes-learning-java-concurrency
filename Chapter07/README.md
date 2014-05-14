ThreadLocal
=========
#1.介绍

ThreadLocal是什么呢？其实ThreadLocal并非是一个线程的本地实现版本，它并不是一个Thread，而是threadlocalvariable(线程局部变量)。也许把它命名为ThreadLocalVar更加合适。ThreadLocal功能非常简单，就是为每一个使用该变量的线程都提供一个变量值的副本，是Java中一种较为特殊的线程绑定机制，是每一个线程都可以独立地改变自己的副本，而不会和其它线程的副本冲突。

从线程的角度看，每个线程都保持一个对其线程局部变量副本的隐式引用，只要线程是活动的并且ThreadLocal实例是可访问的；在线程消失之后，其线程局部实例的所有副本都会被垃圾回收（除非存在对这些副本的其他引用）。

通过ThreadLocal存取的数据，总是与当前线程相关，也就是说，JVM 为每个运行的线程，绑定了私有的本地实例存取空间，从而为多线程环境常出现的并发访问问题提供了一种隔离机制。

ThreadLocal的应用场合，我觉得最适合的是按线程多实例（每个线程对应一个实例）的对象的访问，并且这个对象很多地方都要用到。

ThreadLocal的接口方法：
T get() 
          返回此线程局部变量的当前线程副本中的值。 

protected  T initialValue() 
          返回此线程局部变量的当前线程的“初始值”。 

void remove() 
          移除此线程局部变量当前线程的值。 

void set(T value) 
          将此线程局部变量的当前线程副本中的值设置为指定值。


#2.ThreadLocal的几种误区

最近由于需要用到ThreadLocal，在网上搜索了一些相关资料，发现对ThreadLocal经常会有下面几种误解：

(1)	ThreadLocal是java线程的一个实现
ThreadLocal的确是和java线程有关，不过它并不是java线程的一个实现，它只是用来维护本地变量。针对每个线程，提供自己的变量版本，主要是为了避免线程冲突，每个线程维护自己的版本。彼此独立，修改不会影响到对方。

(2)	ThreadLocal是相对于每个session的

ThreadLocal顾名思义，是针对线程。在java web编程上，每个用户从开始到会话结束，都有自己的一个session标识。但是ThreadLocal并不是在会话层上。其实，Threadlocal是独立于用户session的。它是一种服务器端行为，当服务器每生成一个新的线程时，就会维护自己的ThreadLocal。对于这个误解，个人认为应该是开发人员在本地基于一些应用服务器测试的结果。众所周知，一般的应用服务器都会维护一套线程池，也就是说，对于每次访问，并不一定就新生成一个线程。而是自己有一个线程缓存池。对于访问，先从缓存池里面找到已有的线程，如果已经用光，才去新生成新的线程。所以，由于开发人员自己在测试时，一般只有他自己在测，这样服务器的负担很小，这样导致每次访问可能是共用同样一个线程，导致会有这样的误解：每个session有一个ThreadLocal

(3)	ThreadLocal是相对于每个线程的，用户每次访问会有新的ThreadLocal

理论上来说，ThreadLocal是的确是相对于每个线程，每个线程会有自己的ThreadLocal。但是上面已经讲到，一般的应用服务器都会维护一套线程池。因此，不同用户访问，可能会接受到同样的线程。因此，在做基于TheadLocal时，需要谨慎，避免出现ThreadLocal变量的缓存，导致其他线程访问到本线程变量

(4)	对每个用户访问，ThreadLocal可以多用

可以说，ThreadLocal是一把双刃剑，用得来的话可以起到非常好的效果。但是，ThreadLocal如果用得不好，就会跟全局变量一样。代码不能重用，不能独立测试。因为，一些本来可以重用的类，现在依赖于ThreadLocal变量。如果在其他没有ThreadLocal场合，这些类就变得不可用了。

个人觉得ThreadLocal用得很好的几个应用场合，值得参考
* 存放当前session用户：quake want的jert
* 存放一些context变量，比如webwork的ActionContext
* 存放session，比如Spring hibernate orm的session
