<title>Notification Programming Topics</title>

原文修订日期：2009-08-18

# Introduction #

> Foundation supplies a programming architecture for passing around information about the occurrence of events.

Foundation 为传递有关事件发生的信息提供了一个编程架构。本文描述了该构架的要素并解释如何使用它们。

This document contains the following articles:

- Notifications describes objects that encapsulate information about events.
- Notification Centers describes objects that manage the sending and receiving of notifications.
- Notification Queues describes objects that act as buffers for notification centers.
- Registering for a Notification describes how to register for a notification.
- Posting a Notification describes how to post a notification.
- Delivering Notifications To Particular Threads shows an example of how to handle notifications that should be processed on a particular thread.

# Notifications #

通知封闭了关于事件的信息，如窗口获得了焦点，或网络断开了连接。需要知道某个事件的对象，在通知中心注册说，那个事件发生时它想知道通知。当事件真的发生时，一个通知被 post 到通知中心，后者立即向所有注册对象广播此通知。Optionally, a notification is queued in a notification queue, which posts notifications to a notification center after it delays specified notifications and coalesces notifications that are similar according to some specified criteria you specify.

*Q: 通知机制与控件的事件有何不同？Target-action? `[yourButton addTarget:self action:@selector(methodTouchDown:) forControlEvents:UIControlEventTouchDown];` 哪个更像 Windows Forms 中的控件事件？*

*Target-action 用于 view -> controller 的通信，notification 用于 model 到 controller 的通信。*

## Notifications and Their Rationale ##

> The standard way to pass information between objects is message passing—one object invokes the method of another object. However, message passing requires that the object sending the message know who the receiver is and what messages it responds to. At times, this tight coupling of two objects is undesirable—most notably because it would join together two otherwise independent subsystems. For these cases, a broadcast model is introduced: An object posts a notification, which is dispatched to the appropriate observers through an NSNotificationCenter object, or simply notification center.

在对象间传递信息的标准方式是消息传递 (message passing) ——一个对象调用另一个对象的方法。然而消息传递要求发消息发送者知道谁是接收者、以及接收者可以响应什么消息。有时候不希望两个对象紧密耦合，尤其是会导致两个子系统连接在一起时。在这些情况下，就引入了一个广播模型：一个对象 post 一个通知，通知通过一个 NSNotificationCenter 对象被分派到适当的观察者。

An NSNotification object (referred to as a notification) contains a name, an object, and an optional dictionary. The name is a tag identifying the notification. The object is any object that the poster of the notification wants to send to observers of that notification—typically the object that posted the notification itself. The dictionary may contain additional information about the event.

Notification names: Notification names are typically defined as constant string variables—for example, NSWindowDidBecomeMainNotification. Usually the value of the string is similar to the variable name. You should not, however, use the string value in your code, you should always use the name of the variable—see Registering for Local Notifications for an example.
Many Cocoa frameworks make extensive use of notifications to allow objects to react to events they are interested in. The notifications sent by each class are described in the class’s reference documentation, under the “Notifications” section.

Any object may post a notification. Other objects can register themselves with the notification center as observers to receive notifications when they are posted. The notification center takes care of broadcasting notifications to the registered observers, if any. The object posting the notification, the object included in the notification, and the observer of the notification may all be different objects or the same object. Objects that post notifications need not know anything about the observers. On the other hand, observers need to know at least the notification name and keys to the dictionary if provided.

## Notification and Delegation ##

> Using the notification system is similar to using delegation but has these differences:
>
> - Any number of objects may receive the notification, not just the delegate object. This precludes returning a value.
> - An object may receive any message you like from the notification center, not just the predefined delegate methods.
> - The object posting the notification does not even have to know the observer exists.

通知与委托类似，但有以下不同：

- 接收通知的对象可以有任意多个，而委托只有一个。这样通知的接收者就不能有返回值。
- 对象都可以从通知中心接收任何消息，而委托只能接收预定义的委托方法。
- 发出通知的对象不必知道是否有观察者存在。

# Notification Centers #

通知中心管理消息的收发。通知信息被封闭成 `NSNotification` 对象。发消息和收消息的可以是同一个对象。

Cocoa 包含两种通知中心：

- `NSNotificationCenter` 类管理单个进程内的通知。每个进程都有一个默认的通知中心，可通过 [NSNotificationCenter defaultCenter] 类方法访问。
- `NSDistributedNotificationCenter` 类管理单台计算机内跨越多个进程的通知。每个进程都有一个默认的分布式通知中心，可通过 [NSDistributedNotificationCenter defaultCenter] 类方法访问。
- 要在多台机器间通知，请使用 disbuted objects.

## NSNotificationCenter ##

通知中心以同步的方式向观察者分发通知，即所有的观察者都收到并处理了通知之后，控制才会返回到 poster. 在以异步的方式发送通知，请使用通知队列 (notification queue).

在多线程程序中，通知总是在它被 post 的那个线程中分发出去，这可能与观察者注册时所在的线程不同。

## NSDistributedNotificationCenter ##

Post 分布式的通知是昂贵的操作，通知被发送到一个系统范围内的服务器，后者再分发给注册了分布式通知的所有进程。这一过程中的延迟是没有上界的，如果有太多的通知导致服务器的队列被填满，那么通知可能被丢弃 (dropped).

分布式通知通过进程的 run loop 分发。要接收分布式通知，进程必须以一种 "common" 的模式（如 NSDefaultRunLoopMode）运行 run loop. 若接收通知的进程中多线程的，不要依赖于通知达到主线程。通知一般是被分发到主线程的 run loop 的，但其他线程也可以接收到通知。

> Whereas a regular notification center allows any object to be observed, a distributed notification center is restricted to observing a string object. Because the posting object and the observer may be in different processes, notifications cannot contain pointers to arbitrary objects. Therefore, a distributed notification center requires notifications to use a string as the notification object. Notification matching is done based on this string, rather than an object pointer.

普通的通知中心允许观察任何对象，而分布式通知中心则只允许观察字符串对象。因为通知的发出者和观察者可能位于不同的进程中，通知不能包含指向任意对象的指针，是故分布式通知中心要求使用一个字符串作为通知对象。通知的匹配是基于字符串的，而不是对象指针。