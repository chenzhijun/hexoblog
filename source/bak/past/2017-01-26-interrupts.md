---
layout:     post
title:      "Java并发-线程中断"
subtitle:   "线程中断"
date:       2016-01-26 03:19:00
author:     "chenzhijun"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - Java
    - 并发
---

### Interrupts
============================================================

 

An ___interrupt___ is an indication to a thread that it should stop what it is doing and do something else. It's up to the programmer to decide exactly how a thread responds to an interrupt, but it is very common for the thread to terminate. This is the usage emphasized in this lesson.

中断是停掉线程正在做的事情来做其它事情的一个特征。一个线程怎样回应 interrupt 取决于程序怎样的定义，非常普遍的就是那个线程会停止。这节非常强调这种用法。

A thread sends an interrupt by invoking [interrupt][1] on the Thread object for the thread to be interrupted. For the interrupt mechanism to work correctly, the interrupted thread must support its own interruption.

一个线程发送一个中断通过 Thread 类调用 interrupt 方法来达到中断程序。为了保证终端正确的工作，中断线程必须支持自己的中断。

### Supporting Interruption

How does a thread support its own interruption? This depends on what it's currently doing. If the thread is frequently invoking methods that throw InterruptedException, it simply returns from the run method after it catches that exception. For example, suppose the central message loop in the SleepMessages example were in the run method of a thread's Runnable object. Then it might be modified as follows to support interrupts:

一个线程怎样支持它自己的中断了？这个取决于它当前正在做什么。如果线程频繁的调用出现中断异常的方法，在它抓到异常之后它仅仅会从 run 方法中返回。例如，在 SleepMessages 例子当中假设中间的消息循环在线程的 run 方法中。之后它可以像下面这样的修改来支持终端

```
for (int i = 0; i < importantInfo.length; i++) {
    // Pause for 4 seconds
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        // We've been interrupted: no more messages.
        return;
    }
    // Print a message
    System.out.println(importantInfo[i]);
}
```

Many methods that throw InterruptedException, such as sleep, are designed to cancel their current operation and return immediately when an interrupt is received.

许多方法都可以抛出 InterruptedException ，例如 sleep 方法，被设计成当收到一个中断时，取消其他当前操作并且快速返回

What if a thread goes a long time without invoking a method that throws InterruptedException? Then it must periodically invoke Thread.interrupted, which returns true if an interrupt has been received. For example:

如果一个线程运行了很长时间并且没有抛出 InterruptedException 了？它必须周期性的调用 Thread.interrupted，这回返回 true 如果一个中断被收到时。例如：

```
for (int i = 0; i < inputs.length; i++) {
    heavyCrunch(inputs[i]);
    if (Thread.interrupted()) {
        // We've been interrupted: no more crunching.
        return;
    }
}
```

In this simple example, the code simply tests for the interrupt and exits the thread if one has been received. In more complex applications, it might make more sense to throw an InterruptedException:

在这个简单的例子中，这个代码仅仅只是测试 interrupt 并且出现线程是否接收到中断。在复杂的应用中，它更加有意义来抛出一个中断 InterruptedException:

```
if (Thread.interrupted()) {
    throw new InterruptedException();
}
```

This allows interrupt handling code to be centralized in a catch clause.

允许中断处理代码在 catch 方法里面。

### The Interrupt Status Flag


The interrupt mechanism is implemented using an internal flag known as the interrupt status. Invoking Thread.interrupt sets this flag. When a thread checks for an interrupt by invoking the static method Thread.interrupted, interrupt status is cleared. The non-static isInterrupted method, which is used by one thread to query the interrupt status of another, does not change the interrupt status flag.

中断机制的实现是作为一个内部标识用来作为一个中断状态。调用 Thread.interrupt 设置这个标识。当一个线程通过调用静态方法 Thread.interrupted 来检查中断状态。isInterrupted 是非静态方法，被用来一个线程查询另一个线程的状态，不改变中断状态标识。

By convention, any method that exits by throwing an InterruptedException clears interrupt status when it does so. However, it's always possible that interrupt status will immediately be set again, by another thread invoking interrupt.

安装惯例，任何一个方法通过抛出一个 InterruptedException 异常来退出的都会清除掉中断状态。然而，它也总是在另一个线程调用 interrupt 时，会马上重设中断状态。

--------------------------------------------------------------------------------

via: http://docs.oracle.com/javase/tutorial/essential/concurrency/interrupt.html

[a]:
[1]:https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#interrupt--