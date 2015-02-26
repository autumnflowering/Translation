<title>Local and Remote Notification Programming Guide</title>

Apple official documentation.

原文修订日期：2015.1.24

# About Local Notifications and Remote Notifications #

本地通知和远程通知是两种用户通知，远程通知即所谓的推送通知。两种通知都可以让不在前台 (foreground) 运行的程序通知用户，通知的信息可以消息、即将到来的日历事件、或远程服务器上的新数据。由 OS 呈现后，本地通知和远程通知看起来和听起来并无异样，它们可以显示警告消息或程序图标，同时也可播放一段声音。

用户收到通知后，既可启动程序查阅详情，也可无视之。注意：远程通知和本地通知与广播通知 (`NSNotificationCenter`) 或 KVO 通知无关。

## At a Glance ##

### Local and Remote Notifications Solve Similar Problems ###

任一时刻只能有一个程序在前台活动。许多程序运行在一个基于时间或互连的环境中，程序不在前台时，用户感兴趣的事件也可能发生。本地和远程通知允许在这些事件发生时通知用户。
### Local and Remote Notifications Are Different in Origination ###

本地和远程通知满足不同的设计需要。

- 本地通知由程序自己安排并发送。
- 远程通知，即所谓的推送通知，则来自设备之外。它们源于远程服务器——程序的供应者——并通过 APNs (Apple Push Notification service) 推送到设备上的程序。

### You Register, Schedule, and Handle Both Local and Remote Notifications ###

要让系统在以后的某个时刻分发本地通知，程序注册通知类型（iOS 8 及以后）、使用 `UILocalNotification` 或 `NSUserNotification` 创建一个本地通知对象、指定其分发时间和时间、指定呈现细节、and schedules it for delivery.

要接收远程程序，程序必须注册通知类型，然后向供应者传递一个设备令牌，设备令牌是从 OS 获得的。

OS 分发本地或远程通知、而目标程序又未在前台运行时，它（指 OS）可以通知警告、icon badge number、或声音来向用户呈现通知。If there is a notification alert and the user taps or clicks an action button (or moves the action slider), the app launches and calls a method to pass in the local-notification object or remote-notification payload.

若通知分发时程序正在前台运行，则 app delegate 会收到本地或远程通知。

在 iOS 8 及以后，用户通知可包含自定义行为。而且，基于位置的通知可在用户到达某个特定的地理位置时发出。

### The Apple Push Notification Service Is the Gateway for Remote Notifications ###

APNs 把远程通知传递给有程序注册为接收这些通知的设备。每个设备与 APNs 建立一个可信任的、加密的 IP 连接，并通过该持久连接 (persistent connection) 接收通知。

Apple Push Notification service (APNs) propagates remote notifications to devices having apps registered to receive those notifications. Each device establishes an accredited and encrypted IP connection with the service and receives notifications over this persistent connection. Providers connect with APNs through a persistent and secure channel while monitoring incoming data intended for their client apps. When new data for an app arrives, the provider prepares and sends a notification through the channel to APNs, which pushes the notification to the target device.

### You Must Obtain Security Credentials for Remote Notifications ###

要为远程通知开发及部署 provider side, 必须从 Member Center 获得 SSL 证书。每个证书仅可用于一个程序，通过 bundle ID 识别；证书也仅限于一个或两个环境，一个用于开发，一个用于生产。These environments have their own assigned IP address and require their own certificates. You must also obtain provisioning profiles for each of these environments.

### The Provider Communicates with APNs over a Binary Interface ###

此二进制接口是异步的，使用一个 streaming TCP socket 设计把远程通知内容作为二进制内容发送给 APNs. 开发环境和生产环境有各自独立的接口、地址和端口。对每个接口，都需要使用 TLS 或 SSL, 使用获得的 SSL 证书建立一个安全的通信信道，供应者编撰外发的通知并通过此信道发送给 APNs.

APNs 有一个回馈服务，维护了一个 per-app list of devices for which there were failed-delivery attempts （即 APNs 无法向那个设备上的程序发送远程通知）。供应者应周期性地连接到回馈服务，检查总是推送失败的设备，据此节制向这些设备发送远程通知。

# Local and Remote Notifications in Depth #

本地和远程通知的主要目的都是让程序不在前台运行时也能够通知用户，二者的主要区别很简单：

- 本地通知由程序自己安排，并分发到（与程序）相同的设备。
- 远程通知由供应者的服务器发送给 APNs, 后者再推送到设备。

## Local and Remote Notifications Appear the Same to Users ##

用户可通过以下方式得到通知：

- An onscreen alert or banner
- A badge on the app’s icon
- A sound that accompanies an alert, banner, or badge


## Local and Remote Notifications Appear Different to Apps ##


## More About Local Notifications ##

## More About Remote Notifications ##

# Registering, Scheduling, and Handling User Notifications #

# Apple Push Notification Service #

# Provisioning and Development #

# Provider Communication with Apple Push Notification Service #

# Legacy Information #