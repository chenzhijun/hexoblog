Java 并发
### Lesson: Concurrency

Computer users take it for granted that their systems can do more than one thing at a time. They assume that they can continue to work in a word processor, while other applications download files, manage the print queue, and stream audio. Even a single application is often expected to do more than one thing at a time. For example, that streaming audio application must simultaneously read the digital audio off the network, decompress it, manage playback, and update its display. Even the word processor should always be ready to respond to keyboard and mouse events, no matter how busy it is reformatting text or updating the display. Software that can do such things is known as ___concurrent___ software.

计算机用户经常让他们的系统在同一时间做多件事情。他们希望在文字编辑中还能让下载应用下载文件，并且同时能管理打印队列和流媒体。甚至一个单应用还经常被期待在同一时间做多个事情。例如，一个流媒体应用必须可以同时离线读取数字音频，解压缩它，回滚播放，更新列表。甚至文字处理器应该随时准备回应键盘和鼠标事件，无论它是多忙的在格式化文本或者更新显示。可以这样做的软件被叫做并发软件

The Java platform is designed from the ground up to support concurrent programming, with basic concurrency support in the Java programming language and the Java class libraries. Since version 5.0, the Java platform has also included high-level concurrency APIs. This lesson introduces the platform's basic concurrency support and summarizes some of the high-level APIs in the java.util.concurrent packages.

Java 平台从根本上来设计就是为了支持并发编程，在 Java 编程语言中和 Java 基本类库中拥有基础的并发支持。支持5.0版本之后，Java 平台也有了高级的并发 APIs. 这课主要介绍平台基本的并发支持和一些在 Java java.util.concurrent 包中高级的 APIs 集合。