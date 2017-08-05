Synchronization
============================================================

#

```
Threads communicate primarily by sharing access to fields and the objects reference fields refer to. This form of communication is extremely efficient, but makes two kinds of errors possible: thread interference and memory consistency errors. The tool needed to prevent these errors is synchronization.
```

线程之间的交流主要通过共享域和对象应用。这种交流方式是非常有效的，但是会造成两种可能的错误：线程之间的干扰和内存一致性错误。同步就是解决这些错误的最好工具。
```
However, synchronization can introduce thread contention, which occurs when two or more threads try to access the same resource simultaneously and cause the Java runtime to execute one or more threads more slowly, or even suspend their execution. [Starvation and livelock][6] are forms of thread contention. See the section [Liveness][7] for more information.
```
然而，同步可能产生线程之间的争夺，这可能出现2个或者更多的线程想要同时获取到同一个资源导致 Java 运行时间执行一个或者更多个线程变得更慢，或者挂起他们的执行。饥饿锁是线程争夺的一种形式，详情看 Liveness 章节。
```
This section covers the following topics:
```
这一章包括下面的主题：
```
*   [Thread Interference][1] describes how errors are introduced when multiple threads access shared data.
	[线程干扰] 描述了多个线程同时获取共享资源时出现的错误。
*   [Memory Consistency Errors][2] describes errors that result from inconsistent views of shared memory.
	【内存一致性】描叙了当共享内存不一致的时候的错误结果
*   [Synchronized Methods][3] describes a simple idiom that can effectively prevent thread interference and memory consistency errors.
	【同步方法】描叙了一个简单的方法可以有效的阻止线程互相的干扰和内存一致的错误
*   [Implicit Locks and Synchronization][4] describes a more general synchronization idiom, and describes how synchronization is based on implicit locks.
	【隐藏锁和同步方法】描叙的是另一个常用的同步方法，并且描叙了基于隐藏锁如果同步。
*   [Atomic Access][5] talks about the general idea of operations that can't be interfered with by other threads.
   【原子访问】讲述了一个操作方式可以防止其它线程的干扰。
```
--------------------------------------------------------------------------------

via: 网址




[a]:
[1]:http://docs.oracle.com/javase/tutorial/essential/concurrency/interfere.html
[2]:http://docs.oracle.com/javase/tutorial/essential/concurrency/memconsist.html
[3]:http://docs.oracle.com/javase/tutorial/essential/concurrency/syncmeth.html
[4]:http://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html
[5]:http://docs.oracle.com/javase/tutorial/essential/concurrency/atomic.html
[6]:http://docs.oracle.com/javase/tutorial/essential/concurrency/starvelive.html
[7]:http://docs.oracle.com/javase/tutorial/essential/concurrency/liveness.html
