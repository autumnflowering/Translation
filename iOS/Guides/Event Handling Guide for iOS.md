<title>Event Handling Guide for iOS</title>

原文修订日期：2013-01-28

# Introduction #

iOS 中的事件可以有多种形式，如多点触控事件、运动事件 (motion)，以及控制多媒体的事件，最后一种就是所谓的远程控制事件 (remote control event)，因为它可以来自外部附件。

### UIKit Makes It Easy for Your App to Detect Gestures ###

iOS 程序识别各种触摸并以对用户直观的方式响应它们。有些手势很常见，所以被内置在了 UIKit 中。如 UIButton, UISlider 等 UIControl 的子类，可以响应特定的手势——点击按钮或拖动滑块。配置了这些控件后，它们会在触摸事件发生时向 target 对象发送一个 action 消息。通过 gesture recognizer, 还可以在 view 上使用 target-action 机制。View 被附加了 gesture recognizer 后，整个 view 都表现得像个控件了，它可以响应指定的手势。

Gesture recognizer 为复杂的事件处理逻辑提供了一种更高层的抽象，是处理触摸事件的优选方式，因为它们功能强大、可复用、有适应性。可使用内置的 gesture recognizer 并定制其行为，也可自创 gesture recognizer 以识别新手势。

### An Event Travels Along a Specific Path Looking for an Object to Handle It ###

When iOS recognizes an event, it passes the event to the initial object that seems most relevant for handling that event, such as the view where a touch occurred. If the initial object cannot handle the event, iOS continues to pass the event to objects with greater scope until it finds an object with enough context to handle the event. This sequence of objects is known as a **responder chain**, and as iOS passes events along the chain, it also transfers the responsibility of responding to the event. 

这种设计模式使得事件处理是协作的、动态的。

### A UIEvent Encapsulates a Touch, Shake-Motion, or Remote-Control Event ###

许多事件都是 *UIEvent* 类的实例。Each event object has a type—touch, “shaking” motion, or remote control—and a subtype.

### An App Receives Multitouch Events When Users Touch Its Views ###

> As a rule of thumb, you write your own custom touch-event handling when your app’s response to touch is tightly coupled with the view itself, such as drawing under a touch.

对有些程序来说，使用 UIKit 控件和 gesture recognizer 处理触摸事件已足矣。即使有自定义 view, 还可以使用 gesture recognizer. 一个经验法则是，当对触摸事件的响应与 view 自身紧密耦合时（如触摸绘图），（才需要）编写自定义的触摸事件处理逻辑。 这些情况下，你负责底层的事件处理，你需要实现触摸方法，在其中分析原始的 (raw) 触摸事件并适当地响应。


### An App Receives Motion Events When Users Move Their Devices ###

运动事件提供有关设备位置、方向及移动的信息。加速度计和陀螺仪数据可使你发现设备的倾斜、旋转及摇动 (tilting, rotating, and shaking).

运动事件有不同的形式，你可以使用不同的 framework 处理之。用户摇动设备时，UIKit 会向程序发出一个 UIEvent 对象。要收到高频率、连贯的加速度计和陀螺仪数据，请使用 Core Motion framework.

### An App Receives Remote Control Events When Users Manipulate Multimedia Controls ###

iOS 控制和外部附件向程序发送远程控制事件，这些事件使用户可以控制音视频，比如通过耳机调整音量。

# Gesture Recognizers #

Gesture recognizer 把底层的事件处理代码转换成高层的行为。它们是附加到 view 上的对象，允许 view 以控件的方式响应用户的行为。Gesture regognizer 解析 touches 并判断它们是否对应于某个特定的手势，如 swipe, pinch, or rotation. 如果识别出了所分派的手势，就向 target 对象发送一个 action 消息，target 对象通常是 view's view controller, 对手势作出响应。这种设计模式简单有效，可动态地决定某个 view 响应何种用户行为，而且无须子类化 view 即可向它添加 gesture recognizer.

![A gesture recognizer attached to a view](images/gesture_recognizer_role.png)

## Use Gesture Recognizers to Simplify Event Handling ##

### Built-in Gesture Recognizers Recognize Common Gestures ###

### Gesture Recognizers Are Attached to a View ###

### Gestures Trigger Action Messages ###

## Responding to Events with Gesture Recognizers ##

## Defining How Gesture Recognizers Interact ##

## Gesture Recognizers Interpret Raw Touch Events ##

## Regulating the Delivery of Touches to Views ##

## Creating a Custom Gesture Recognizer ##


# Event Delivery: The Responder Chain #

# Multitouch Events #

# Motion Events #

# Remote Control Events #
