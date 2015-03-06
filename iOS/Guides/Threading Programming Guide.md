<title>Threading Programming Guide</title>

Apple official documentation.

原文修订日期：2014.07.15

# Introduction #

线程是在单个程序内并发地执行多个代码路径的技术之一。尽管较新的技术，如 operation 对象和 Grand Central Dispatch (GCD) 为实现并发提供了更现代、更有效的基础设施，但 OS X 和 iOS 还提供了创建及管理线程的接口。

重要：若开发新程序，则鼓励研究实现并发的其他备选技术，尚未熟悉实现多线程程序所需的技术时尤为如此。这些备选技术简化了实现并发所需的工作，可提供比传统的线程更好的性能。欲知详情，请参阅 Concurrency Programming Guide.

## Organization of This Document ##

This document has the following chapters and appendixes:

- About Threaded Programming introduces the concept of threads and their role in application design.
- Thread Management provides information about the threading technologies in OS X and how you use them.
- Run Loops provides information about how to manage event-processing loops in secondary threads.
- Synchronization describes synchronization issues and the tools you use to prevent multiple threads from corrupting data or crashing your program.
- Thread Safety Summary provides a high-level summary of the inherent thread safety of OS X and iOS and some of their key frameworks.

## See Also ##

For information about the alternatives to threads, see Concurrency Programming Guide.

本文仅概述了 POSIX 线程 API 的使用，关于 POSIX 例程的详情，请查阅 pthread man page. 更深入的讲解请查阅 Programming with POSIX Threads, 作者 David R. Butenhof. 

# About Threaded Programming #

TBC...

# Thread Management #

TBC...

# Run Loops #

Run loops are part of the fundamental infrastructure associated with threads. A run loop is an event processing loop that you use to schedule work and coordinate the receipt of incoming events. Run loop 的目的是，有任务时使线程忙碌，无任务时使之睡眠。

Run loop 的管理不是完全自动的，你仍需设计线程的代码以在适当的时机启动 run loop, 以及对输入事件作出响应。Cocoa 和 Core Foundation 都提供了 run loop objects, 助你配置及管理线程的 run loop.程序无须显式创建这些对象，每个线程——包括主线程——都有一个相关联的 run loop 对象。只有辅助 (secondary) 线程需要显式地运行其 run loop, the app frameworks 会自动在主线程上建立并运行 run loop, 这是程序启动过程的一部分。

NSRunLoop, CFRunLoop class ref.

## Anatomy of a Run Loop ##

物如其名，run loop 是线程进入其中并用以运行事件处理程序的一个 loop. 你的代码提供用以实现实际循环部分的控制语句，即驱动 run loop 的 `while` 或 `for` 循环。在此循环中使用 run loop 对象“运行”事件处理代码。 

> A run loop is very much like its name sounds. It is a loop your thread enters and uses to run event handlers in response to incoming events. Your code provides the control statements used to implement the actual loop portion of the run loop—in other words, your code provides the while or for loop that drives the run loop. Within your loop, you use a run loop object to "run” the event-processing code that receives events and calls the installed handlers.

Run loop 从两种来源接收事件：

- Input sources 以异步方式发出事件，通常是来自其他线程或程序的消息。
- Timer sources 以同步方式发出事件，在预定的时刻或以重复的间隔发生。

Both types of source use an application-specific handler routine to process the event when it arrives.

Figure 3-1 shows the conceptual structure of a run loop and a variety of sources. The input sources deliver asynchronous events to the corresponding handlers and cause the runUntilDate: method (called on the thread’s associated NSRunLoop object) to exit. Timer sources deliver events to their handler routines but do not cause the run loop to exit.

![Structure of a run loop and its sources](images/runloop_structure.jpg)

### Run Loop Modes ###

### Input Sources ###

### Timer Sources ###

### Run Loop Observers ###

### The Run Loop Sequence of Events ###

## When Would You Use a Run Loop? ##

## Using Run Loop Objects ##

## Configuring Run Loop Sources ##

# Synchronization #

# Appendix A: Thread Safety Summary #
