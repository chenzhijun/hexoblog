Defining and Starting a Thread
============================================================

An application that creates an instance of Thread must provide the code that will run in that thread. There are two ways to do this

一个应用如果要创建一个 Thread 实例必须提供重写线程的 run 方法。这里提供两种方式来实现：

*   Provide a Runnable object. The [Runnable][1] interface defines a single method, run, meant to contain the code executed in the thread. The Runnable object is passed to the Thread constructor, as in the [HelloRunnable][2] example：
	提供一个 Runnable 类。Runable 接口只定义了一个方法，run，意味着包含执行的代码在线程里面。Runable 类时传递给了 Thread 的构造方法例如细面的 HelloRuannable 实例：

```
public class HelloRunnable implements Runnable {
    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new Thread(new HelloRunnable())).start();
    }

}
```
*   Subclass Thread. The Thread class itself implements Runnable, though its run method does nothing. An application can subclass Thread, providing its own implementation of run, as in the[HelloThread][3] example:
	继承 Thread。Thread 类本身继承了 Runable, 尽管它的 run 方法不做任何事。一个应用可以使 Thread 的子类，提供它自己的 run 方法实现。如下面的例子：

```
public class HelloThread extends Thread {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new HelloThread()).start();
    }

}
```

Notice that both examples invoke Thread.start in order to start the new thread.

注意：这两个例子的都必须由 Thread.start 来启动一个新线程。

Which of these idioms should you use? The first idiom, which employs a Runnable object, is more general, because the Runnable object can subclass a class other than Thread. The second idiom is easier to use in simple applications, but is limited by the fact that your task class must be a descendant of Thread. This lesson focuses on the first approach, which separates the Runnable task from the Thread object that executes the task. Not only is this approach more flexible, but it is applicable to the high-level thread management APIs covered later.

你会使用哪一种方式了？第一种方式，提供一个 Runable 类，更加普遍。因为这个 Runnable 类可以继承除了 Thread 类之外的其它类。第二种方式更加简单的使用在一些简单的实例上面，但是它限制了你的任务类必须是 Thread 类的子类。本节主要是集中在第一种方式，这种从 Thread 类分开 Runnable 任务来执行任务的方式。不仅仅这种方式更加灵活，更多的是在之后的高级线程 APIs 管理中更加适用。

The Thread class defines a number of methods useful for thread management. These include static methods, which provide information about, or affect the status of, the thread invoking the method. The other methods are invoked from other threads involved in managing the thread and Thread object. We'll examine some of these methods in the following sections.

Thread 类定义了一系列有用的线程管理方法。这些包括静态方法，提供信息的方法，或者影响状态，或者线程的调用。一些调用其它线程的方法管理线程和 Thread 类。我们会检验一些里面的方法在下面的章节。

--------------------------------------------------------------------------------

[a]:
[1]:https://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html
[2]:http://docs.oracle.com/javase/tutorial/essential/concurrency/examples/HelloRunnable.java
[3]:http://docs.oracle.com/javase/tutorial/essential/concurrency/examples/HelloThread.java
