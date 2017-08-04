Pausing Execution with Sleep
============================================================


Thread.sleep causes the current thread to suspend execution for a specified period. This is an efficient means of making processor time available to the other threads of an application or other applications that might be running on a computer system. The sleep method can also be used for pacing, as shown in the example that follows, and waiting for another thread with duties that 
are understood to have time requirements, as with the SimpleThreads example in a later section.

Thread.sleep 可以让当前线程在指定的时间段内暂停执行。这是一个有效的方式使得处理器时间可以让应用或者其它应用的其它线程可以得到执行。sleep 方法也可以是一种减速执行方法，例如下面的例子所示，或者义务的等待其它具有时间要求的线程，稍候将在线程实例中提到：

Two overloaded versions of sleep are provided: one that specifies the sleep time to the millisecond and one that specifies the sleep time to the nanosecond. However, these sleep times are not guaranteed to be precise, because they are limited by the facilities provided by the underlying OS. Also, the sleep period can be terminated by interrupts, as we'll see in a later section. In any case, you cannot assume that invoking sleep will suspend the thread for precisely the time period specified.

提供两个重载版本的 sleep 方法:一个指定了 sleep 的毫秒时间，一个制定了 sleep 的纳秒时间。然而，这些 sleep 的时间不能准备的得到保证，因为他们受限于运行底层操作系统的设备。因此，sleep 的时间段可以被中断停止。如我们在下一节看到的。在任何情况下，你都不可以假定 sleep 方法的调用会是一个恰好的准确时间。

The [SleepMessages][1] example uses sleep to print messages at four-second intervals:

SleepMessages 实例使用了 sleep 方法来答应 messages 每隔四秒：

```
public class SleepMessages {
    public static void main(String args[])
        throws InterruptedException {
        String importantInfo[] = {
            "Mares eat oats",
            "Does eat oats",
            "Little lambs eat ivy",
            "A kid will eat ivy too"
        };

        for (int i = 0;
             i < importantInfo.length;
             i++) {
            //Pause for 4 seconds
            Thread.sleep(4000);
            //Print a message
            System.out.println(importantInfo[i]);
        }
    }
}
```

Notice that main declares that it throws InterruptedException. This is an exception that sleep throws when another thread interrupts the current thread while sleep is active. Since this application has not defined another thread to cause the interrupt, it doesn't bother to catch InterruptedException.

注意到主线程中声明了它必须抛出 InterruptedException。这是一个在 sleep 方法运行时被其他线程打断时抛出的异常。尽管这个程序没有定义其它线程来导致终端，它也不能阻止抓获 InterruptedException.

--------------------------------------------------------------------------------


[a]:
[1]:http://docs.oracle.com/javase/tutorial/essential/concurrency/examples/SleepMessages.java