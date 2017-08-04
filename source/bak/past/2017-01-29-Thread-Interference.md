---
layout:     post
title:      "Java并发-线程干扰"
subtitle:   "线程干扰"
date:       2016-01-29 03:19:00
author:     "chenzhijun"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - Java
    - 并发
---

### Thread Interference
============================================================

Consider a simple class called [Counter][1]

参考一个简单的Counter类

```
class Counter {
    private int c = 0;

    public void increment() {
        c++;
    }

    public void decrement() {
        c--;
    }

    public int value() {
        return c;
    }

}
```

Counter is designed so that each invocation of increment will add 1 to c, and each invocation of decrement will subtract 1 from c. However, if a Counter object is referenced from multiple threads, interference between threads may prevent this from happening as expected.

Counter 被设计成每一次增加调用就给 C 加1，每一次减少调用就给 C 减1.然而，如果一个 Counter 类是多线程的一个引用，线程间的干扰就会阻止预期结果。

Interference happens when two operations, running in different threads, but acting on the same data, interleave. This means that the two operations consist of multiple steps, and the sequences of steps overlap.

干扰发生在两个正在运行在不同的线程中交错的操作同一个数据。这说明着两个操作由多个步骤组成，并且步骤之间还有重叠。

It might not seem possible for operations on instances of Counter to interleave, since both operations on c are single, simple statements. However, even simple statements can translate to multiple steps by the virtual machine. We won't examine the specific steps the virtual machine takes — it is enough to know that the single expression c++ can be decomposed into three steps:

它看起来不太可能出现 Counter 的实例操作会交错，因为两个对 c 的操作都是单独的简单的语句。然而，就算简单的语句也可以被虚拟机当做是多个步骤执行。我们不需要去检查虚拟机花费的具体的步骤-表达式 c++ 就足够我们知道可以分解成三个步骤：

1.  Retrieve the current value of c.
	 获取 c 的值。
2.  Increment the retrieved value by 1.
	 在获取的值上加1
3.  Store the incremented value back in c.
	 保存增加后的值给 c

The expression c-- can be decomposed the same way, except that the second step decrements instead of increments.

表达式 c-- 也可以用相同的方式分解，只是第二步用减法代替了加法。

Suppose Thread A invokes increment at about the same time Thread B invokes decrement. If the initial value of c is 0, their interleaved actions might follow this sequence:

在线程 B 调用了减的方法的同时，假设线程 A 调用了加的方法。如果初始化 c 的值是0，他们交叉执行之后可能出现下面的循序：

1.  Thread A: Retrieve c.
    线程 A：获取 c 的值
2.  Thread B: Retrieve c.
    线程 B：获取 c 的值
3.  Thread A: Increment retrieved value; result is 1.
    线程A：获取的值+1，结果是1.
4.  Thread B: Decrement retrieved value; result is -1.
    线程B：获取的值-1，结果是-1.
5.  Thread A: Store result in c; c is now 1.
	线程 A：存储 c 的值，c 现在是1.
6.  Thread B: Store result in c; c is now -1.
   线程 B：存储 c 的值，c 现在是-1.

Thread A's result is lost, overwritten by Thread B. This particular interleaving is only one possibility. Under different circumstances it might be Thread B's result that gets lost, or there could be no error at all. Because they are unpredictable, thread interference bugs can be difficult to detect and fix.

线程 A 的结果丢失，因为被线程 B 重写了结果。这个特别的交叉顺序只是可能的一种。在不同的情况下它也有可能是线程 B 的结果丢失，或者这里也有可能没有错误。因为他们是不可预期的，线程干扰的 bugs 是非常难以检查和修复的。

--------------------------------------------------------------------------------

via: 网址


[a]:
[1]:http://docs.oracle.com/javase/tutorial/essential/concurrency/examples/Counter.java