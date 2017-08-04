Thread Objects

Each thread is associated with an instance of the class **Thread**. There are two basic strategies for using Thread objects to create a concurrent application.

每一个线程都与 **Thread** 类的实例相关。这里有两种方式使用 Thread 类来创建多线程应用。

* To directly control thread creation and management, simply instantiate Thread each time the application needs to initiate an asynchronous task.
 如果是想要直接控制线程的创建和管理，在每次应用需要初始化一个异步的任务时，实例化一次 Thread 类，

* To abstract thread management from the rest of your application, pass the application's tasks to an executor.
如果要在应用中抽象化线程管理，只要将应用的任务传递给执行程序。

This section documents the use of Thread objects. Executors are discussed with other high-level concurrency objects.

这个章节的文档是 Thread 类的使用，Executors 类会在另一个高等级高级别的并发类里面讨论。