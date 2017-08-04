---
layout:     post
title:      "Java并发-进程和线程"
subtitle:   "进程和线程"
date:       2016-01-22 03:19:00
author:     "chenzhijun"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - Java
    - 并发
---

### Processes and Threads

In concurrent programming, there are two basic units of execution: processes and threads. In the Java programming language, concurrent programming is mostly concerned with threads. However, processes are also important.

在并发编程中，这里有两个基本的执行单元：进程和线程。在 Java 编程语言中，并发编程主要关注的是线程。然而，进程一样的非常重要。

A computer system normally has many active processes and threads. This is true even in systems that only have a single execution core, and thus only have one thread actually executing at any given moment. Processing time for a single core is shared among processes and threads through an OS feature called time slicing.

一个计算机系统平常有许多活动的进程和线程。在系统中仅仅只有一个单个的执行主线程，在任何个点的时候仅仅只有一个线程在真的执行。单个主进程运行时间是和所有进程和线程通过一个操作系统的特点，这个特点就是时间切片（时间轮询）。

It's becoming more and more common for computer systems to have multiple processors or processors with multiple execution cores. This greatly enhances a system's capacity for concurrent execution of processes and threads — but concurrency is possible even on simple systems, without multiple processors or execution cores.

现在对于计算机系统拥有多处理器和多个具有执行核心的处理器越来越平常了。它极大的增加了系统的容量对于进程和线程的并发执行，但是并发即使在没有多核处理器和主执行的简单系统上也是可以的。

Processes

A process has a self-contained execution environment. A process generally has a complete, private set of basic run-time resources; in particular, each process has its own memory space.

一个进程拥有独有的执行环境。一个进程通常有一个完整的，私有的一系列运行资源。在特定情况下，每一个进程拥有它的单独内存空间。

Processes are often seen as synonymous with programs or applications. However, what the user sees as a single application may in fact be a set of cooperating processes. To facilitate communication between processes, most operating systems support ___Inter Process Communication___ (IPC) resources, such as pipes and sockets. IPC is used not just for communication between processes on the same system, but processes on different systems.

进程通常也被认为和程序或者应用相同。然而，让用户见到的一个小应用可能实际上就是一组互相合作的进程集。为了促进进程之间的通信，大多数操作系统支持内部进程间通信资源。例如管道和通口。IPC 不仅仅只是用来在同一个系统中进程之间互相通讯，进程也可以在不同的系统上进行通信。

Most implementations of the Java virtual machine run as a single process. A Java application can create additional processes using a ___ProcessBuilder___ object. Multiprocess applications are beyond the scope of this lesson.

大多数 Java 虚拟机的实现方式都是运行一个单进程。一个 Java 应用可以创建其它的进程使用<strong>ProcessBuilder</strong>类。多进程应用已经超出了这节课的范围。

Threads

Threads are sometimes called lightweight processes. Both processes and threads provide an execution environment, but creating a new thread requires fewer resources than creating a new process.

线程同时又被称为是轻量级的进程。进程和线程都提供一个执行环境，但是创建一个新线程比创建一个新进程所需要的资源少很多。

Threads exist within a process — every process has at least one. Threads share the process's resources, including memory and open files. This makes for efficient, but potentially problematic, communication.

线程出现在进程里面 - 每一个进程至少拥有一个线程。线程共享了进程的资源，包括内存和打开的文件。这是为了有效并且解决了一个潜在的问题：通信。

Multithreaded execution is an essential feature of the Java platform. Every application has at least one thread — or several, if you count "system" threads that do things like memory management and signal handling. But from the application programmer's point of view, you start with just one thread, called the main thread. This thread has the ability to create additional threads, as we'll demonstrate in the next section.

多线程执行是 Java 平台的一个必要的特征。每一个应用都至少拥有一个线程或者一系列线程，如果你统计过那些可以做内存管理或者信号处理的“system”线程。但是从应用程序员的角度来看，你开启的只是一个线程，称作主线程。这个线程有能力创建其它的线程，我们将在下个章节演示。




