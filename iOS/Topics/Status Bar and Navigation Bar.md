<title>Status Bar and Navigation Bar</title>

iOS 7 的状态栏是透明的。

iOS 6 和 iOS 7 中默认的状态栏和导航栏。

![Default status bar and navigation bar changes across iOS 6 and iOS 7](images/default_status_bar_and_nav_bar_changes_in_ios7.jpg)


# Navigation Bar #

## Background Color ##

In iOS 7, the tintColor property is no longer used for setting the color of the bar. Instead, use the barTintColor property to change the background color. You can insert the below code in the didFinishLaunchingWithOptions: of AppDelegate.m.

iOS 7 不再使用 `tintColor` 属性设置（导航？）栏的颜色，而用 `barTintColor` 属性修改其背景色。

``` Objective-C
// App Delegate, didFinishLaunchingWithOptions:
[[UINavigationBar appearance] setBarTintColor:[UIColor yellowColor]];
```

效果：

![Changing the Background Color of the Navigation Bar](images/changing_nav_bar_bg_color.jpg)

By default, the `translucent` property of navigation bar is set to `YES`. Additionally, there is a system blur applied to all navigation bars. Under this setting, iOS 7 tends to desaturate the color of the bar. Here are the sample navigation bars with different translucent setting.
可通过 Attributes Inspector 修改导航栏的 `translucent` 属性。

![](images/navigation_bar_translucent.jpg)


## Background Image ##
If your app uses a custom image as the background of the bar, you’ll need to provide a “taller” image so that it extends up behind the status bar. The height of navigation bar is changed from 44 points (88 pixels) to 64 points (128 pixels).
要为导航栏设置背景图，请使用一个略高的图像，这样它会向上延伸到状态栏的堆叠层次（Z 轴）之下，这样导航栏的高度也从 44 点增加到了 64 点（因为状态栏的高度是 20 点）。

``` Objective-C
[[UINavigationBar appearance] setBackgroundImage:[UIImage imageNamed:@"nav_bg.png"] forBarMetrics:UIBarMetricsDefault];
```

![](images/navigation_bar_background_image.jpg)

# References #

[http://www.appcoda.com/customize-navigation-status-bar-ios-7/](http://www.appcoda.com/customize-navigation-status-bar-ios-7/)

http://possiblemobile.com/2013/09/developers-guide-to-the-ios-7-status-bar/

http://stackoverflow.com/questions/19063365/how-to-change-the-status-bar-background-color-and-text-color-on-ios-7