Memory Consistency Errors
============================================================

# 

Memory consistency errors occur when different threads have inconsistent views of what should be the same data. The causes of memory consistency errors are complex and beyond the scope of this tutorial. Fortunately, the programmer does not need a detailed understanding of these causes. All that is needed is a strategy for avoiding them.

内存一致性错误发生在不同的线程对于同一个数据见到的不一样的结果。这回导致复杂的内存一致性错误，这已经超过了本教程的范围。幸运的是，程序开发者不需要详细的清楚这种发生原因。仅仅需要的是知道如何避免他们的策略。

The key to avoiding memory consistency errors is understanding the happens-before relationship. This relationship is simply a guarantee that memory writes by one specific statement are visible to another specific statement. To see this, consider the following example. Suppose a simple int field is defined and initialized:

避免内存一致性错误的关键在于理解发生前的关系。这个关系是保证内存只被一个特定的语句写入，并且这个语句对其它语句是可见的。知道了这个，考虑下西面的例子。假设定义了一个简单的 int 域，并且已经初始化。

```
int counter = 0;
```

The counter field is shared between two threads, A and B. Suppose thread A increments counter:

counter 变量被两个线程共享，A 和 B。假设 A 是增加 counter。

```
counter++;
```

Then, shortly afterwards, thread B prints out counter:
然后，里面线程 B 输出 counter。

```
System.out.println(counter);
```

If the two statements had been executed in the same thread, it would be safe to assume that the value printed out would be "1". But if the two statements are executed in separate threads, the value printed out might well be "0", because there's no guarantee that thread A's change to counter will be visible to thread B — unless the programmer has established a happens-before relationship between these two statements.

如果两个语句在同一个线程里面被执行，它可能是我们假定的值输出“1”.但是如果这两个语句在不同的线程里面执行，这个值输出的时候可能为“0”，因为这是没有保证的在 A 改变了 counter 的值并且对 B 可见。除非这个程序员建立了一个“发生前”关系在两个语句之间。

There are several actions that create happens-before relationships. One of them is synchronization, as we will see in the following sections.
这里有很多种方式来创建”发生前（happens-before）“ 关系。一种方式就是同步，我们将在下面的部分中见到的一样。

We've already seen two actions that create happens-before relationships.

我们已经知道了两个创建“happens-before"关系的途径。

*   When a statement invokes Thread.start, every statement that has a happens-before relationship with that statement also has a happens-before relationship with every statement executed by the new thread. The effects of the code that led up to the creation of the new thread are visible to the new thread.
    当一个语句调用 Thread.start 方法，每一个语句就有一个 happens-before 关系和其它被新县城执行的的任何一条语句。受影响的创建一个新线程代码对于新线程是可见的。

*   When a thread terminates and causes a Thread.join in another thread to return, then all the statements executed by the terminated thread have a happens-before relationship with all the statements following the successful join. The effects of the code in the thread are now visible to the thread that performed the join.
    当一个线程中止并且导致了一个 Thread.join 在其它的线程中返回。被中止的线程的所有的执行的语句都有一个 happens-before 关系和所有成功的 join 后的语句。在线程中的这段受影响的代码对于 join 的代码是可见的。

For a list of actions that create happens-before relationships, refer to the [Summary page of the java.util.concurrent package.][1].

想要了解创建 happens-before 关系的列表动作，查看 java.util.concurrent 包。

--------------------------------------------------------------------------------

via: 网址


[a]:
[1]:https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html#MemoryVisibility