Grand Central Dispatch Tutorial for Swift

Source: http://www.raywenderlich.com/79149/grand-central-dispatch-tutorial-swift-part-1

Learn about concurrency in this Grand Central Dispatch in-depth tutorial series.

Translation: http://www.cocoachina.com/swift/20150129/11057.html

## Getting Started ##

GCD is the marketing name for **libdispatch**, Apple’s library that provides support for concurrent code execution on multicore hardware on iOS and OS X. It offers the following benefits:

- GCD can improve your app’s responsiveness by helping you defer computationally expensive tasks and run them in the background.
- GCD provides an easier concurrency model than locks and threads and helps to avoid concurrency bugs.
- 通过推迟计算量大的任务并将其置于后台，从而提高程序的响应力。
- 提供了一个比锁和线程简单的并发模型，利于避免并发 bug.

To understand GCD, you need to be comfortable with several concepts related to threading and concurrency. These can be both vague and subtle, so take a moment to review them briefly in the context of GCD.

### Serial vs. Concurrent ###

These terms describe when tasks are executed with respect to each other. Tasks executed serially are always executed one at a time. Tasks executed concurrently might be executed at the same time.

### Tasks ###
For the purposes of this tutorial you can consider a task to be a closure. In fact, you can also use GCD with function pointers, but in most cases this is substantially more tricky to use. Closures are just easier!

在本教程是可认为任务是闭包。实际上也可在 GCD 中使用函数指针，但闭包更简单。

Don’t know what a closure is in Swift? Closures are self-contained, callable blocks of code that can be stored and passed around. When called, they behave like functions and can have parameters and return values. In addition a closure “captures” variables it uses from outside its own scope — that is, it sees the variables from the enclosing scope and remembers their value.

Swift closures are similar to Objective-C blocks and they are almost entirely interchangeable. Their only limitation is just that you cannot, from Objective-C, interact with Swift closures that expose Swift-only language features, like tuples. But interacting with Objective-C from Swift is unhindered, so whenever you read documentation that refers to an Objective-C block then you can safely substitute a Swift closure.

Swift 的闭包类似于 Objective-C 中的块，二者几乎可互换，但闭包可使用 Swift 语言特有的特性，如 tuple.

### Synchronous vs. Asynchronous ###

These terms describe when a function will return control to the caller, and how much work will have been done by that point.

同步和异步描述函数何时把控制权返回给调用者，以及返回时完成了多少工作。

A synchronous function returns only after the completion of a task that it orders.
An asynchronous function, on the other hand, returns immediately, ordering the task to be done but not waiting for it. Thus, an asynchronous function does not block the current thread of execution from proceeding on to the next function.

Be careful — when you read that a synchronous function “blocks” the current thread, or that the function is a “blocking” function or blocking operation, don’t get confused! The verb blocks describes how a function affects its own thread and has no connection to the noun block, which describes an anonymous function literal in Objective-C. Also keep in mind that whenever the GCD documentation refers to Objective-C blocks it is interchangeable with Swift closures.

### Critical Section ###

This is a piece of code that must not be executed concurrently, that is, from two threads at once. This is usually because the code manipulates a shared resource such as a variable that can become corrupt if it’s accessed by concurrent processes.

### Race Condition ###

This is a situation where the behavior of a software system depends on a specific sequence or timing of events that execute in an uncontrolled manner, such as the exact order of execution of the program’s concurrent tasks. Race conditions can produce unpredictable behavior that aren’t immediately evident through code inspection.

### Deadlock ###

Two (or sometimes more) items — in most cases, threads — are said to be deadlocked if they all get stuck waiting for each other to complete or perform another action. The first can’t finish because it’s waiting for the second to finish. But the second can’t finish because it’s waiting for the first to finish.

### Thread Safe ###

Thread safe code can be safely called from multiple threads or concurrent tasks without causing any problems (data corruption, crashing, etc). Code that is not thread safe must only be run in one context at a time. An example of thread safe code is let a = ["thread-safe"]. This array is read-only and you can use it from multiple threads at the same time without issue. On the other hand, an array declared with var a = ["thread-unsafe"] is mutable and can be modified. That means it’s not thread-safe since several threads can access and modify the array at the same time with unpredictable results. Variables and data structures that are mutable and not inherently thread-safe should only be accessed from one thread at a time.

线程安全的代码可在多个线程或并发任务中安全地调用，而不会引起任何问题（数据损坏、崩溃等）。非线程安全的代码在某一时刻只能运行在一个上下文中。线程安全的代码如 `let a = ["thread-safe"]`. 该数组是只读的，可在多个线程中同时使用它而不会引起问题。而 `var a = ["thread-unsafe"]` 定义的是可变数组，意味着它不是线程安全的，因为多个线程可以同时修改这个数组，导致不可知的结果。变量以及可变且固有非线程安全的数据结构，同一时刻只应在一个线程内访问。

### Context Switch ###

A context switch is the process of storing and restoring execution state when you switch between executing different threads on a single process. This process is quite common when writing multitasking apps, but comes at a cost of some additional overhead.

### Concurrency vs Parallelism ###

Separate parts of concurrent code can be executed “simultaneously”. However, it’s up to the system to decide how this happens — or if it happens at all.

Multi-core devices execute multiple threads at the same time via parallelism; however, in order for single-cored devices to achieve this, they must run a thread, perform a context switch, then run another thread or process. This usually happens quickly enough as to give the illusion of parallel execution as shown by the diagram below:

多核设备可通过并行在同一时刻执行多个线程；而单核设备只能通过上下文切换来执行多个线程或进程，上下文切换的速度很快，给人并行执行的假象。

![Parallelism vs. Concurrency](images/Concurrency_vs_Parallelism.png)

Although you may write your code to use concurrent execution under GCD, it’s up to GCD to decide how much parallelism is required. Parallelism requires concurrency, but concurrency does not guarantee parallelism.

尽管可用 GCD 编写并发执行的代码，但由 GCD 决定着需要多大程序的并行。并行需要并发，而并发却不保证并行。

The deeper point here is that concurrency is actually about structure. When you code with GCD in mind, you structure your code to expose the pieces of work that can run simultaneously, as well as the ones that must not be run simulataneously. If you want to delve more deeply into this subject, check out this excellent talk by Rob Pike.

## Queues ##

GCD provides dispatch queues to handle submitted tasks; these queues manage the tasks you provide to GCD and execute those tasks in FIFO order. This guarantees that the first task added to the queue is the first task started in the queue, the second task added will be the second to start, and so on down the line.

GCD 提供了调度队列来处理提交的任务，它们管理你提交给 GCD 的任务并以 FIFO 的顺序执行之。

All dispatch queues are themselves thread-safe in that you can access them from multiple threads simultaneously. The benefits of GCD are apparent when you understand how dispatch queues provide thread-safety to parts of your own code. The key to this is to choose the right kind of dispatch queue and the right dispatching function to submit your work to the queue.

所有的调度队列自身都是线程安全的，可在多个线程中同时访问它们。理解了调度队列如何为你的代码提供线程安全性时，GCD 的优点就很明显了。关键是选择正确的调度队列和调度函数来提交你的任务。

### Serial Queues ###

Tasks in serial queues execute one at a time, each task starting only after the preceding task has finished. As well, you won’t know the amount of time between one task ending and the next one beginning, as shown in the diagram below:

一次只执行一个任务。你无法知道一个任务结束和下一个任务开始前会间隔多少时间。

![Serial Queue](images/serial_queues.png)

The execution timing of these tasks is under the control of GCD; the only thing you’re guaranteed to know is that GCD executes only one task at a time and that it executes the tasks in the order they were added to the queue.

执行的时序由 GCD 控制，你只知道 GCD 一次只执行一个任务，顺序是 FIFO.

Since no two tasks in a serial queue can ever run concurrently, there is no risk they might access the same critical section at the same time; that protects the critical section from race conditions with respect to those tasks only. So if the only way to access that critical section is via a task submitted to that dispatch queue, then you can be sure that the critical section is safe.

串行队列中的任务不会同时访问同一个临界区，使该临界区不会遭受来自这些任务的竞态条件。

### Concurrent Queues ###

Tasks in concurrent queues are guaranteed to start in the order they were added… and that’s about all you’re guaranteed! Items can finish in any order and you have no knowledge of the time it will take for the next task to start, nor the number of tasks that are running at any given time. Again, this is entirely up to GCD.

并发队列中的任务也是以 FIFO 的顺序开始的，这是唯一可以保证的。任务结束的顺序不可预知，下一个任务开始的时间亦不可预知，某一时刻有多少个任务在运行亦不可知，这完全取决于 GCD.

The diagram below shows a sample task execution plan of four concurrent tasks under GCD:

![Concurrent Queue](images/concurrent_queue.png)

Notice how Task 1, 2, and 3 all ran quickly, one after another, while it took a while for Task 1 to start after Task 0 started. Also, Task 3 started after Task 2 but finished first.

The decision of when to start a task is entirely up to GCD. If the execution time of one task overlaps with another, it’s up to GCD to determine if it should run on a different core, if one is available, or instead to perform a context switch to a different task.

Just to make things interesting, GCD provides you with at least five particular queues to choose from within each queue type.

## Queue Types ##

First, the system provides you with a special serial queue known as the main queue. Like any serial queue, tasks in this queue execute one at a time. However, it’s guaranteed that all tasks will execute on the main thread, which is the only thread allowed to update your UI. This queue is the one to use for sending messages to UIView objects or posting notifications.

系统提供了一个特殊的**串行**队列，即所谓的主队列，其中所有的任务都会在主线程中执行——主线程是唯一一个允许更新 UI 的线程。

The system also provides you with several concurrent queues. These queues are linked with their own Quality of Service (QoS) class. The QoS classes are meant to express the intent of the submitted task so that GCD can determine how to best prioritize it:

- QOS_CLASS_USER_INTERACTIVE: The user interactive class represents tasks that need to be done immediately in order to provide a nice user experience. Use it for UI updates, event handling and small workloads that require low latency. The total amount of work done in this class during the execution of your app should be small.
- QOS_CLASS_USER_INITIATED: The user initiated class represents tasks that are initiated from the UI and can be performed asynchronously. It should be used when the user is waiting for immediate results, and for tasks required to continue user interaction.
- QOS_CLASS_UTILITY: The utility class represents long-running tasks, typically with a user-visible progress indicator. Use it for computations, I/O, networking, continous data feeds and similar tasks. This class is designed to be energy efficient.
- QOS_CLASS_BACKGROUND: The background class represents tasks that the user is not directly aware of. Use it for prefetching, maintenance, and other tasks that don’t require user interaction and aren’t time-sensitive.

Be aware that Apple’s APIs also uses the global dispatch queues, so any tasks you add won’t be the only ones on these queues.

系统还提供了 4 个全局的**并发**队列，它们各自关联了一个 QoS 等级。QoS 等级表述任务的意图，以便 GCD 决定其优先级。
注意 Apple 的 API 也使用全局队列，因此这些队列中并非只有你提交的任务。

Finally, you can also create your own custom serial or concurrent queues. That means you have at least five queues at your disposal: the main queue, four global dispatch queues, plus any custom queues that you add to the mix!

还可以创建自定义的串行或并发队列。

And that’s the big picture of dispatch queues!

The “art” of GCD comes down to choosing the right queue dispatching function to submit your work to the queue. The best way to experience this is to work through the examples below, where we’ve provided some general recommendations along the way.

GCD 的“艺术”可归结为选择正确的队列调度函数来提交任务。

## Singletons and Thread Safety ##

Singletons. Love them or hate them, they’re as popular in iOS as cats are on the web. :]
One frequent concern with singletons is that often they’re not thread safe. This concern is well-justified given their use: singletons are often used from multiple controllers accessing the singleton instance at the same time. The PhotoManager class is a singleton, so you will need to consider this issue.

单例通常不是线程安全的，比如多个控制器同时访问一个单例。

There are two cases to consider, thread-safety during initialization of the singleton instance and during reads and writes to the instance.

有两种情形需要考虑，在单例初始化过程中和被读写过程中的线程安全性。

Let us consider initialization first. This turns out to be the easy case, because of how Swift initializes variables at global scope. In Swift, global variables are initialized when they are first accessed, and they are guaranteed to be initialized in an atomic fashion. That is, the code performing initialization is treated as a critical section and is guaranteed to complete before any other thread gets access to the global variable. How does Swift do this for us? Behind the scenes, Swift itself is using GCD, by using the function dispatch_once, as discussed in this Swift Blog post.

首先说初始化。由于 Swift 在全局作用域内初始化变量的方式，这是较简单的情形。Swift 中的全局变量在其首次使用时被初始化，且保证以原子方式初始化，即执行初始化的代码被视为临界区。在幕后，Swift 是使用 GCD 中的 `dispatch_once` 实现这一点的。详见[该博文](http://https://developer.apple.com/swift/blog/?id=7)。

dispatch_once executes a closure once and only once in a thread safe manner. If one thread is in the middle of executing the critical section — the task passed to dispatch_once — then other threads will just block until it completes. And once it does it complete, then those threads and other threads will not execute the section at all. And by defining the singleton as a global constant with let, we can further guarantee the variable will never be changed after initialization. In a sense, all Swift global constants are naturally born as singletons, with thread-safe initialization.

`dispatch_once` 以线程安全的方式执行且仅执行一次闭包。若一个线程正在临界区内执行——即传递给 `dispatch_once` 的任务——此时其他的线程将会阻塞，直到它完成，这样其他线程就不会再执行这一临界区了。通过 let 把单例定义为全局常量，可进一步保证变量在初始化后不会被修改。从某种意义上说，所有的 Swift 全局常量天生都是单例，且以线程安全的方式初始化。

But we still need to consider reads and writes. While Swift uses dispatch_once to ensure that we are initializing the singleton in a thread-safe manner, it does not make the data type it represents thread safe. For example if the global variable is an instance of a class, you could still have critical sections within the class that manipulate internal data. Those would need to be made thread safe in other ways, such as by synchronizing access to the data, as you’ll see in the following sections.

但还需考虑读和写。尽管 Swift 使用 `dispatch_once` 保证单例以线程安全的方式初始化，但这不能保证它所表示的数据类型也是线程安全的。如全局变量是一个类实例，在类中仍可以有操纵内部数据的临界区。这就需要用其他方式来达到线程安全了，such as by synchronizing access to the data.

## Handling the Readers and Writers Problem ##

Thread-safe instantiation is not the only issue when dealing with singletons. If a singleton property represents a mutable object, like the photos array in PhotoManager, then you need to consider whether that object is itself thread-safe.

实例化线程安全性不是单例的唯一问题。若单例属性表示的是可变 (mutable) 对象，那就要考虑该对象自身是否是线程安全的。

In Swift any variable declared with the let keyword is considered a constant and is read-only and thread-safe. Declare the variable with the var keyword however, and it becomes mutable and not thread-safe unless the data type is designed to be so. The Swift collection types like Array and Dictionary are not thread-safe when declared mutable. What about Foundation containers like NSArray? Are they thread safe? The answer is — “probably not”! Apple maintains a helpful list of the numerous Foundation classes which are not thread-safe.

Swift 中用 let 关键字声明的是常量，也是只读和线程安全的；用 var 声明的是可变的，不是线程安全的——除非该数据类型被设计成线程安全的。数据和字典等集合类型，被声明为可变时，就不是线程安全的。那么 Foundation 中如 NSArray 之类的容器呢？它们是线程安全的吗？——或许不是。Apple 维护了一个 Foundation 类的列表[列表](http://https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Multithreading/ThreadSafetySummary/ThreadSafetySummary.html)，其中的类不是线程安全。

Although many threads can read a mutable instance of Array simultaneously without issue, it’s not safe to let one thread modify the array while another is reading it. Your singleton doesn’t prevent this condition from happening in its current state. To see the problem, have a look at addPhoto in PhotoManager.swift, which has been reproduced below:
