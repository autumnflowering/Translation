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

- 本地通知由程序自己安排 (schedule) 并发送。
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
- A badge on the app's icon
- A sound that accompanies an alert, banner, or badge

Badge number 指的是主屏上程序图标右上角显示的红色小数字。

Users can also selectively enable or disable remote notification types (that is, icon badging, alert messages, and sounds) for specific apps.

## Local and Remote Notifications Appear Different to Apps ##

若本地或远程通知到达时目标程序位于最前端，则 app delegate 的 `application:didReceiveRemoteNotification:` or `application:didReceiveLocalNotification:` 方法会被调用。

若程序不在最前端或未运行，则通过检查传递给 `application:didFinishLaunchingWithOptions:` 的 options 字典中的 `UIApplicationLaunchOptionsLocalNotificationKey` or `UIApplicationLaunchOptionsRemoteNotificationKey` 来处理消息。

## More About Local Notifications ##

本地通知适于基本时间的行为，如日历或待办事项程序。Apps that run in the background for the limited period allowed by iOS might also find local notifications useful. For example, apps that depend on servers for messages or data can poll their servers for incoming items while running in the background; if a message is ready to view or an update is ready to download, they can handle the data as needed, and notify users if appropriate.

本地通知是 `UILocalNotification` or `NSUserNotification` 的实例，大致有 3 类属性：

- Scheduled time. 必须指定 OS 发出通知的日期和时间，即所谓的 **fire date**. 可以为 fire date 指定时区，还可要求 OS 以一定的间隔重复地发出通知（如每周、每月等）。 
- Notification type. These properties include the alert message, the title of the default action button, the app icon badge number, a sound to play, and optionally in iOS 8 and later, a category of custom actions.
- Custom data. Local notifications can include a user info dictionary of custom data.

一个设备上的一个程序至多可有 64 个预定的 (scheduled) 本地通知，超过此限制的通知会被丢弃，只保留最近要发生的 64 个通知。复发性 (recurring) 通知被视为一个通知。

> Each app on a device is limited to 64 scheduled local notifications. The system discards scheduled notifications in excess of this limit, keeping only the 64 notifications that will fire the soonest. Recurring notifications are treated as a single notification.

## More About Remote Notifications ##

iOS 或 Mac 程序通常是一个基于客户端/服务端模型的较大程序的一部分，客户端安装在设备上，服务端主要为客户端程序提供数据，故被称为供应者 (provider). 客户端周期性地连接到供应者并下载需要的数据。邮件和社交程序用的都是这种 C/S 模型。

如果供应者有新数据时，程序未连接到供应者、或者根本未运行呢？它如何得知有新数据？答案是远程（推送）通知。远程通知是供应者发送给 OS 的一个短消息，OS 再通知客户端程序的用户。APNs 是远程通知中的主要技术。

远程通知的目的类似于桌面系统上的后台程序，但无须额外开销。对于未正在运行的程序——对 iOS 来说，未在前台运行的程序——通知是间接地发生的。OS 代替程序接收远程通知并通知用户，用户启动程序，后者再从供应者下载数据。若通知到来时程序正在运行，则它可以直接处理之。

顾名思义，APNs 使用 a remote design 来向设备发送远程通知。Push design 与 pull design 的不同在于，通知的受众被动地监听、而不是主动地轮询。Push design 使信息传播更广泛、更及时，且比 pull design 稳定。APNs 使用一个持久的 IP 实现远程通知。

多数远程通知会包含一个有效载荷 (payload): a property list containing APNs-defined properties specifying how the user is to be notified. 由于性能的原因，有效载荷故意较小。尽管可自定义有效载荷的属性，但永远不要将远程通知机制用于数据传输，因为远程通知的分发不能得到保证。

APNs 保留从供应者收到的最后一个通知，设备上线且未收到此通知时，APNs 就将所保存的通知推送给它。iOS 设备可通过 Wi-Fi 和蜂窝数据网络接收远程通知，OS X 计算机通过 Wi-Fi 和以太网接收。

重要：在 iOS 中，仅当无蜂窝数据连接或设备是 iPad Tpuch 时，才会使用 Wi-Fi 接收远程通知。对于有些设备，要通过 Wi-Fi 接收通知，其屏幕必须亮着（即设备未睡眠）或必须连接了电源。而 iPad 即使在睡眠时仍会连着 Wi-Fi AP, 故仍可接收远程通知。The Wi-Fi radio wakes the host processor for any incoming traffic. 过于频繁地发送通知不利于设备的电源续航，因为设备必须访问网络才能接收通知。

为程序添加远程通知特性需要从 Member Center 为 iOS Developer Program or Mac Developer Program 获得正确的证书，并为客户端和服务端编写代码。Provisioning and Development explains the provisioning and setup steps, and Provider Communication with Apple Push Notification Service and Scheduling, Registering, and Handling Notifications describe the details of implementation.

# Registering, Scheduling, and Handling User Notifications #

## Registering for Notification Types in iOS ##

iOS 8 及以后，使用本地或远程通知的程序必须注册欲发送的通知的类型，这样系统就可以让用户限制程序可以显示的通知类型。若某个通知类型未为某个程序启用，则系统不会显示相应类型的通知——即使在通知有效载荷中有指定。

在 iOS 中使用 UIApplication 的 `registerUserNotificationSettings:` 方法注册通知类型，通知类型表示程序收到通知时所显示的 UI 元素：badging the app's icon, 播放一段声音、显示一个警告。若不注册通知类型，则系统会静默地把所有远程通知都推送给程序，*静默*即不显示任何 UI. 

``` Objective-C
UIUserNotificationType types = UIUserNotificationTypeBadge | UIUserNotificationTypeSound | UIUserNotificationTypeAlert;
UIUserNotificationSettings *mySettings = [UIUserNotificationSettings settingsForTypes:types categories:nil];
[[UIApplication sharedApplication] registerUserNotificationSettings:mySettings];
```

Notice the use of the categories parameter above. A category is a group of actions that can be displayed in conjunction with a single notification. You can learn more about categories in Using Notification Actions in iOS.

首次调用 `registerUserNotificationSettings:` 方法时，iOS 会显示一个对话框，询问用户允许程序显示所注册的哪些通知；用户回复后，iOS 异步地回调 app delegate 的 `application:didRegisterUserNotificationSettings:` 方法，传递一个 `UIUserNotificationType` 对象，指定用户允许的通知类型。用户可能会更改一开始的设置，故准备展示通知时请调用  `currentUserNotificationSettings`.

在 iOS 8 及以后，调用 `registerUserNotificationSettings:` 既适于本地通知，也适于远程通知，于是 `registerForRemoteNotificationTypes:` 在 iOS 8 中就废弃了。

## Scheduling Local Notifications ##

In iOS, you create a `UILocalNotification` object and schedule its delivery using the `scheduleLocalNotification:` method of UIApplication.
In OS X, you create an `NSUserNotification` object (which includes a delivery time) and the `NSUserNotificationCenter` is responsible for delivering it appropriately. (An OS X app can also adopt the `NSUserNotificationCenterDelegate` protocol to customize the behavior of the default `NSUserNotificationCenter` object.)

在 iOS 中创建及 schedule 本地通知的步骤：

1. 在 iOS 8 及以后，注册通知类型。在 OS X 及较早版本的 iOS 中，只需为远程通知注册类型。若已注册了通知类型，请调用 `currentUserNotificationSettings` 以获得用户接受的通知类型。
1. 分配并初始化一个 `UILocalNotification` 对象。
1. 设置 OS 应发出通知的日期和时间，即 `fireDate` 属性。 
If you set the timeZone property to the NSTimeZone object for the current locale, the system automatically adjusts the fire date when the device travels across (and is reset for) different time zones. (Time zones affect the values of date components—that is, day, month, hour, year, and minute—that the system calculates for a given calendar and date value.)
You can also schedule the notification for delivery on a recurring basis (daily, weekly, monthly, and so on).
1. As appropriate, configure an alert, icon badge, or sound so that the notification can be delivered to users according to their preferences. （欲知何种通知适于何种情形，请参阅 iOS Human Interface Guidelines 中的 Notifications 一节）
	- An alert has a property for the message (the alertBody property) and for the title of the action button or slider (alertAction). Specify strings that are localized for the user’s current language preference. If your notifications can be displayed on Apple Watch, assign a value to the alertTitle property.
	- To display a number in a badge on the app icon, use the `applicationIconBadgeNumber` property.
	- To play a sound, assign a sound to the `soundName` property. You can assign the filename of a nonlocalized custom sound in the app’s main bundle or you can assign `UILocalNotificationDefaultSoundName` to get the default system sound. A sound should always accompany the display of an alert message or the badging of an icon; a sound should not be played in the absence of other notification types. 声音总应与其他通知类型一起出现，而不能单独行动。
1. Optionally, you can attach custom data to the notification through the `userInfo` property. For example, a notification that’s sent when a CloudKit record changes includes the identifier of the record, so that a handler can get the record and update it.
1. Optionally, in iOS 8 and later, your local notification can present custom actions that your app can perform in response to user interaction, as described in Using Notification Actions in iOS.
1. Schedule the local notification for delivery.
You schedule a local notification by calling `scheduleLocalNotification:`. The app uses the fire date specified in the `UILocalNotification` object for the moment of delivery. Alternatively, you can present the notification immediately by calling the `presentLocalNotificationNow:` method.

The method in Listing 2-2 creates and schedules a notification to inform the user of a hypothetical to-do list app about the impending due date of a to-do item. There are a couple things to note about it. For the alertBody, alertAction, and alertTitle properties, it fetches from the main bundle (via the NSLocalizedString macro) strings localized to the user’s preferred language. It also adds the name of the relevant to-do item to a dictionary assigned to the userInfo property.



## Registering for Remote Notifications ##

## Handling Local and Remote Notifications ##

## Using Notification Actions in iOS ##

## Using Location-Based Notifications ##

## Preparing Custom Alert Sounds ##

## Passing the Provider the Current Language Preference (Remote Notifications) ##

# Apple Push Notification Service #

# Provisioning and Development #

# Provider Communication with Apple Push Notification Service #

# Legacy Information #