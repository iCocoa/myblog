---
title: Local and Remote Notification Programming Guide 译文
date: 2017-11-27 20:26:21
tags: iOS
---


原文地址：[本地和远程通知编程指南](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/)

# 应用中的通知

## 本地和远程通知概览

> 重要 </br>
> 这篇文档包含开发中有关 API 或技术的初步信息，这些信息可能会改变，并且根据这篇文档来实现的软件应当在最终的操作系统软件中进行测试。

本地通知和远程通知是在应用有新数据可用时通知用户的两种方式，即使此时应用不在前台运行。例如，短信应用可能会让用户知道有新的短信来了，日历应用可能会通知用户即将到来的约会。本地通知和远程通知的区别很简单：

* 对于本地通知，应用在本地配置通知的细节并把这些细节传给系统，然后由系统来处理通知的传递（当应用不在前台时）。iOS、tvOS、watchOS 都支持本地通知。
* 对于远程通知，使用公司服务器中的一个通过`苹果推送通知服务`把数据推送到用户的设备。iOS、tvOS、watchOS、macOS 都支持远程通知。

本地通知和远程通知都需要添加代码来支持应用中的通知的调度和处理。对于远程通知，必须提供一个服务器环境，该环境能够接收来自用户设备的数据和发送通知相关的数据到 `苹果推送消息服务` (简称 APNs，由苹果提供的用来处理远程通知传递的服务)。

### User Notifications 和 User Notifications UI 框架

从 iOS 10、watchOS 3、tvOS 10 开始，User Notifications 框架提供一致的方式来和处理本地通知。除了管理本地通知，该框架也支持远程通知的处理，然而远程通知的配置仍然需要一些平台特有的 API。因为这是一个独立的框架，所以可以在应用中或者扩展中使用，比如 WatchKit 扩展。

> 注意</br>
> macOS 上远程通知的配置和处理需要使用平台特有的方法（在 AppKit 框架中找）

User Notifications 框架也支持创建 *通知服务应用扩展* (notification service app extension)，它可以让你在远程通知传递之前修改通知的内容。如果在应用中包含通知服务应用扩展，系统会把收到的通知在传递给用户之前先传递给扩展。可以使用这类扩展来给应用的通知实现端到端的加密、在通知传递前修改其内容，又或者下载与通知相关的额外的图片或媒体文件。

User Notifications UI 框架是 User Notifications 的配套，它可以让你自定义系统的通知界面的外观。使用User Notifications UI 框架来定义 *通知内容应用扩展*(notification content app extension)，它的任务就是提供一个包含自定义内容的视图控制器来显示在通知界面中。系统会显示自定义视图控制器而不是默认的系统界面。可以使用这种扩展在通知界面中加入多媒体或动态内容。

更多有关 User Notifications 框架的类的信息，请看 [User Notifications Framework Reference](https://developer.apple.com/documentation/usernotifications)。关于创建通知内容应用扩展的类的信息，请看 [User Notifications UI Framework Reference](https://developer.apple.com/documentation/usernotificationsui)。

### 什么时候使用本地通知和远程通知

因为 iOS、tvOS 和 watchOS 上的应用并非一直处于运行状态，所以当应用有新的信息需要呈现出来时，本地通知提供一种方式来提醒用户。比如，一个在后台从服务器拉取数据的应用可以在收到一些有趣的信息时调度一个本地通知。本地通知也很适合类似日历和待办事项清单的应用，这些应用需要在一个特定的时间或者抵达特定的地点时提醒用户。

当应用的部分或全部数据都由公司服务器来管理时，远程通知很合适。使用远程，由你决定何时推送通知给用户设备。例如，一个短信应用可能会使用远程通知让用户知道何时有新的短信到来。因为通知从你的服务器发出，所以你可以在任意时刻发送远程通知，包括应用不在运行状态的时候。

### 在用户看来，本地通知和远程通知是一样的

对用户来说，在设备上呈现的远程通知和本地通知是没有区别的。两者都有着由系统提供的同样的默认外观。某些情况下，你可以自定义外观，但大部分情况下你只是选择你想要的通知用户的方式。具体地说，你可以从以下选项中选出一种来传递通知：

* 屏幕上弹框或横幅
* 应用图标角标
* 伴随弹框，横幅或者角标一起的声音

当配置本地和远程通知时，为要传递的通知选择一种最合适的交互类型。比如，一个代办事项清单应用可能有很多条目，其中每一条目都有完成的时间和优先级。对于高优先级的事项，当经过完成时间点时，也许需要弹框提醒用户立刻处理这一事项。对于低优先级的事项，也许采用更微妙的方式来提醒用户完成该事项，一个应用角标或者播放一段声音。

弹框把消息直接显示给用户，但角标和提示音的含义由取决于应用。可以使用不同的提示音来传达特定类型的事件，比如消息来了或者任务完成了。角标包含一个数值，该数值通常用来表明等候用户注意的事项的数目。图 1-1 显示了 iOS 应用图标上的角标位置。

**图 1-1** 一个包含角标数字的应用图标

<img style='width:80px;height:80px' src="https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/badged_app_2x.png" />

<!--![图 1-1](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/badged_app_2x.png)-->

为了避免惹恼用户，常常明智而审慎地使用本地和远程通知。系统允许用户根据每个应用来启用和禁用弹框、声音以及角标的呈现。虽然通知依然会传递到应用，但是系统仅仅会根据当前的启用选项来通知用户。如果用户完全禁用通知，APNs 不会往用户的设备传递应用的通知并且本地通知的调度总会失败。

## 管理应用的通知支持

应用必须在启动的时候进行配置以支持本地和远程通知。具体地说，必须提前进行配置，如果需要做以下事情：

* 通知到来时显示弹框、播放声音、显示角标
* 随着通知到来显示自定义动作按钮

通常，应当在应用程序完成启动之前完成所有配置。在 iOS 和 tvOS 中，这意味着通知的支持配置不能晚于 `UIApplication` 的代理方法 [application:didFinishLaunchingWithOptions:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622921-application)。在 watchOS 中，配置支持不能晚于 `WKExtension` 的代理方法 [applicationDidFinishLaunching](https://developer.apple.com/documentation/watchkit/wkextensiondelegate/1628241-applicationdidfinishlaunching)。你或许会在这之后进行配置，但必须要避免在配置完成之前调度任何指向应用的本地或远程通知。

支持远程通知的应用需要额外的配置，详见 [Configuring Remote Notification Support](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1)。

### 请求用户授权

在 iOS、tvOS、watchOS 中，应用必须得到授权才能在收到通知时显示弹框、播放声音或显示角标。请求授权把这些交互的控制权交到用户手里，用户可以同意或者拒绝请求，稍后用户也可以在系统设置里面修改应用的授权设置。

调用 `UNUserNotificationCenter` 共享对象的 [requestAuthorizationWithOptions:completionHandler: ](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649527-requestauthorization) 方法请求授权。如果所有请求的交互类型都得到授权，系统调用完成处理块并传入 `granted` 参数，值为 `YES`。如果有一个及以上的交互类型被拒绝，该参数的值为 `NO`。清单 2-1 演示了如何请求播放声音和显示弹框授权。根据交互类型是否得到授权，使用完成处理块来更新应用的行为。

清单 2-1 请求用户交互授权

```
OBJECTIVE-C

UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert + UNAuthorizationOptionSound)
   completionHandler:^(BOOL granted, NSError * _Nullable error) {
      // Enable or disable features based on authorization.
}];


SWIFT

let center = UNUserNotificationCenter.current()
center.requestAuthorization(options: [.alert, .sound]) { (granted, error) in
    // Enable or disable features based on authorization.
}
```

应用首次启动并调用 [requestAuthorizationWithOptions:completionHandler:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649527-requestauthorization) 方法时，系统会提示用户选择同意或拒绝请求的交互。因为系统会保存用户的选择，所以应用之后启动时再调用这个方法不会再次提醒用户。

> 注意</br>
> 用户可以随时在系统设置中修改应用的授权的交互类型。为了精确判定可以使用的交互类型，可以调用 `UNUserNotificationCenter` 的 [getNotificationSettingsWithCompletionHandler: ](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649524-getnotificationsettingswithcompl) 方法。

### 配置分类和可操作通知

可操作通知给用户提供一种快速、简单的方式来执行相关任务以回应通知。可操作通知的界面会显示用户可以点击的自定义动作按钮，而不是强制用户启动应用。每一个按钮被点击后都会隐藏通知界面并把选中的动作转发给应用立即处理，这可以免去用户打开应用并进一步操作的麻烦，因此更省时。

应用必须显式地为可操作通知添加支持。在应用启动时，必须注册一个或多个分类（分类定义了应用发送的通知的类型）。每个分类都关联了该类通知传递时用户可以执行的操作。每个分类可以有多达 4 个关联动作，然而显示的动作数量通常取决于通知显示的方式和位置。例如，横幅显示的动作不超过两个。

> 注意</br>
> 可操作通知仅支持 iOS 和 watchOS。

#### 为应用注册通知分类

分类定义了应用支持的通知类型并向系统传达想要的通知呈现方式。使用分类给通知关联自定义动作并指定该类通知的处理方式选项。例如，使用分类选项来指定通知是否可以被显示在 CarPlay 环境中。

在应用启动时，使用 [UNUserNotificationCenter](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter) 对象的 [setNotificationCategories:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649512-setnotificationcategories) 的方法立即注册应用的所有分类。在调用该方法前，创建一个或多个  [UNNotificationCategory](https://developer.apple.com/documentation/usernotifications/unnotificationcategory) 类的实例并指定在该类型的通知显示时使用的分类名和选项。分类名只在应用内部，用户永远看不到。在调度通知时，把分类名放到通知负载里面，系统会利用分类名来取出选项和显示通知。

清单 2-2 演示了如何创建一个简单的 [UNNotificationCategory](https://developer.apple.com/documentation/usernotifications/unnotificationcategory) 对象并向系统注册。这个分类名为 “GENERAL” 并配置了一个自定义关闭动作选项，这使得系统会在用户关闭通知界面时通知应用。

清单 2-2 创建和注册通知分类

```
OBJECTIVE-C

UNNotificationCategory* generalCategory = [UNNotificationCategory
     categoryWithIdentifier:@"GENERAL"
     actions:@[]
     intentIdentifiers:@[]
     options:UNNotificationCategoryOptionCustomDismissAction];
 
// Register the notification categories.
UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center setNotificationCategories:[NSSet setWithObjects:generalCategory, nil]];


SWIFT

let generalCategory = UNNotificationCategory(identifier: "GENERAL",
                                             actions: [],
                                             intentIdentifiers: [],
                                             options: .customDismissAction)
 
// Register the category.
let center = UNUserNotificationCenter.current()
center.setNotificationCategories([generalCategory])

```

不要求为所有调度的通知指定一个分类，但是，不包含分类的通知在显示时没有任何自定义动作或配置选项。

#### 给分类添加自定义动作

每个注册的分类可能包含多达 4 个自定义动作。当分类包含自定义动作时，系统会在通知界面添加按钮，每个按钮上都带有自定义动作的标题。如果用户点击自定义动作，系统会发送相应的动作标识符到应用，并根据需要启动应用。

通过创建一个 [UNNotificationAction](https://developer.apple.com/documentation/usernotifications/unnotificationaction) 对象并把它加到分类对象中来定义一个自定义动作。每个动作都包含相应按钮的标题以及如何显示按钮和处理关联任务的选项。当用户选中一个动作，系统会为应用提供动作的标识符字符串（这字符串可以用来识别要执行的任务）。清单 2-3 在清单 2-2 例子的基础上添加了一个包含 2 个自定义动作的新分类。

清单 2-3 为分类定义自定义动作

```
OBJECTIVE-C

UNNotificationCategory* generalCategory = [UNNotificationCategory
      categoryWithIdentifier:@"GENERAL"
      actions:@[]
      intentIdentifiers:@[]
      options:UNNotificationCategoryOptionCustomDismissAction];
 
// Create the custom actions for expired timer notifications.
UNNotificationAction* snoozeAction = [UNNotificationAction
      actionWithIdentifier:@"SNOOZE_ACTION"
      title:@"Snooze"
      options:UNNotificationActionOptionNone];
 
UNNotificationAction* stopAction = [UNNotificationAction
      actionWithIdentifier:@"STOP_ACTION"
      title:@"Stop"
      options:UNNotificationActionOptionForeground];
 
// Create the category with the custom actions.
UNNotificationCategory* expiredCategory = [UNNotificationCategory
      categoryWithIdentifier:@"TIMER_EXPIRED"
      actions:@[snoozeAction, stopAction]
      intentIdentifiers:@[]
      options:UNNotificationCategoryOptionNone];
 
// Register the notification categories.
UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center setNotificationCategories:[NSSet setWithObjects:generalCategory, expiredCategory,
      nil]];
      
      
SWIFT

let generalCategory = UNNotificationCategory(identifier: "GENERAL",
                                             actions: [],
                                             intentIdentifiers: [],
                                             options: .customDismissAction)
 
// Create the custom actions for the TIMER_EXPIRED category.
let snoozeAction = UNNotificationAction(identifier: "SNOOZE_ACTION",
                                        title: "Snooze",
                                        options: UNNotificationActionOptions(rawValue: 0))
let stopAction = UNNotificationAction(identifier: "STOP_ACTION",
                                      title: "Stop",
                                      options: .foreground)
 
let expiredCategory = UNNotificationCategory(identifier: "TIMER_EXPIRED",
                                             actions: [snoozeAction, stopAction],
                                             intentIdentifiers: [],
                                             options: UNNotificationCategoryOptions(rawValue: 0))
 
// Register the notification categories.
let center = UNUserNotificationCenter.current()
center.setNotificationCategories([generalCategory, expiredCategory])
```

虽然可以给每个分类指定多达 4 个自定义动作，但系统在某些情况下可能只显示前面 2 个。例如，当在横幅中显示通知时仅显示两个动作。当初始化  [UNNotificaitonCategory](https://developer.apple.com/documentation/usernotifications/unnotificationcategory) 对象时，常常设置动作数组以便最相关的动作就是数组的首个元素。

如果使用 [UNTextInputNotificationAction](https://developer.apple.com/documentation/usernotifications/untextinputnotificationaction) 类来设置动作，系统会提供一种方式来让用户输入文本（作为通知响应的一部分）。在向用户收集自由格式的文本时，文本输入动作很有用。例如，一个短信应用可能允许用户为短信提供自定义回复。当文本输入动作传递到应用中处理时，系统会把用户的回复打包到一个 [UNTextInputNotificationResponse](https://developer.apple.com/documentation/usernotifications/untextinputnotificationresponse) 对象中。

有关如何处理自定义动作被选中的信息，请看 [Responding to the Selection of a Custom Action](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW2)。

### 准备自定义提示音

本地和远程通知可以指定在通知传递时播放的自定义提示音。可以把音频数据打包到一个 `aiff`、`wav` 或 `caf` 文件中。因为音频文件由系统声音设备播放，所以自定义的的音频必须是以下音频数据格式中的一种：

* Linear PCM
* MA4 (IMA/ADPCM)
* µLaw
* aLaw

把自定义的音频文件放到应用包中或应用容器目录的 `Library/Sounds` 文件夹中。自定义的音频播放时长必须在 30s 以内，如果超过 30s，改为播放默认的系统声音。

可以使用 `afconvert` 工具来转换音频。例如，要把 16 bit 的 PCM 系统音频 `Submarine.aiff` 转换为 IMA4 音频保存到一个 CAF 文件中，可以在终端使用以下命令：

`afconvert /System/Library/Sounds/Submarine.aiff ~/Desktop/sub.caf -d ima4 -f caff -v`

如何关联音频文件到通知的信息，请看 [Adding a Sound to the Notification Content](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW3)。

### 管理应用的通知设置

因为用户可以随时修改应用的通知设置，所以你可以随时使用 `UNUserNotificationCenter` 共享对象的 [getNotificationSettingsWithCompletionHandler:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649524-getnotificationsettingswithcompl) 方法来获取应用的授权状态。该方法返回一个 `UNNotificationSettings ` 对象，它的内容反映应用当前的授权状态和当前的通知环境。

使用 [UNNotificationSettings](https://developer.apple.com/documentation/usernotifications/unnotificationsettings) 对象中的信息来调整应用中与通知相关的代码。把应用的弹框、角标、声音的授权设置传达给 `提供者`，以便它调整远程通知负载中包含的选项。(提供者即由开发者部署、管理并配置以和 APNs 一起工作的服务器，有关提供者的更多信息，请看 [APNs Overview](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1))。开发者可以使用其他设置来调整通知的调度和配置，有关更多可用的设置的信息，请看 [UNNotificationSettings Class Reference](https://developer.apple.com/documentation/usernotifications/unnotificationsettings) 。

### 管理通知传递

当本地和远程通知不直接由应用或用户处理时，它们会被显示在通知中心中以便稍后查看。使用 `UNUserNotificationCenter` 共享对象的 [getDeliveredNotificationsWithCompletionHandler:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649520-getdeliverednotificationswithcom) 方法去获取仍显示在通知中心中的通知的列表。如果发现有任何已过期的、不该再显示给用户看的通知，可以使用 [removeDeliveredNotificationsWithIdentifiers:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649500-removedeliverednotificationswith) 方法移除。

## 调度和处理本地通知

当应用不在运行状态时，本地通知为此提供一种方式来提醒用户。本地通知在某一时刻（应用在前台或后台时）调度。在调度之后，系统负责在合适的时间把通知传递给用户。在系统传递通知时应用不需要处于运行状态。

如果应用不在运行状态（或者处于后台），系统直接把本地通知显示给用户。系统可以通过提示框或横幅、使用声音或者显示应用角标等来提醒用户。如果应用提供了通知内容应用扩展，系统甚至可以使用自定义界面来提醒用户。如果通知到来时应用处于前台，系统会给机会应用在内部处理通知。

>注意</br>
>本地通知只支持 iOS、watchOS 和 tvOS，在 macOS 上，当应用处于后台时不需要本地通知来显示角标、播放声音或显示弹框，这些特性在 AppKit 框架中已经有支持。

### 配置本地通知

配置一个本地通知有以下步骤：

1. 使用通知的细节来创建配置一个 [UNMutableNotificationContent](https://developer.apple.com/documentation/usernotifications/unmutablenotificationcontent)  对象；
2. 创建 [UNCalendarNotificationTrigger](https://developer.apple.com/documentation/usernotifications/uncalendarnotificationtrigger)、[UNTimeIntervalNotificationTrigger](https://developer.apple.com/documentation/usernotifications/untimeintervalnotificationtrigger) 或者  [UNLocationNotificationTrigger](https://developer.apple.com/documentation/usernotifications/unlocationnotificationtrigger)  对象来描述通知传递的触发条件；
3. 使用内容和触发器信息来创建一个 [UNNotificationRequest](https://developer.apple.com/documentation/usernotifications/unnotificationrequest) 对象；
4. 调用 [addNotificationRequest:withCompletionHandler:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649508-add) 方法来调度通知，请看 [Scheduling Local Notifications for Delivery](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW5)。

当为通知创建内容时，填充 `UNMutableNotificationContent ` 对象的属性（反映了期望的用户交互类型）。例如，当想要显示一个提示框时填充 [title](https://developer.apple.com/documentation/usernotifications/unmutablenotificationcontent/1649858-title) 和 [body](https://developer.apple.com/documentation/usernotifications/unmutablenotificationcontent/1649874-body) 属性。系统根据你提供的属性来决定如何与用户交互。你也可以在处理传递到应用中的本地通知时使用这个对象中的数据。

创建完通知内容之后，创建一个触发器对象来规定何时传递通知。User Notificaiton 框架提供基于时间和基于位置两种触发器。使用自己需要的条件来配置触发器，然后用它和内容来创建 `UNNotificationRequest` 对象。

清单 3-1 演示了如何创建和配置与闹铃相关的本地通知。使用 `UNCalendarNotificationTrigger ` 促使通知在指定的日期或时间传递，例子中是下次时钟经过早上 7:00 时。

清单 3-1 创建和配置一个本地通知

```
OBJECTIVE-C

UNMutableNotificationContent* content = [[UNMutableNotificationContent alloc] init];
content.title = [NSString localizedUserNotificationStringForKey:@"Wake up!" arguments:nil];
content.body = [NSString localizedUserNotificationStringForKey:@"Rise and shine! It's morning time!"
        arguments:nil];
 
// Configure the trigger for a 7am wakeup.
NSDateComponents* date = [[NSDateComponents alloc] init];
date.hour = 7;
date.minute = 0;
UNCalendarNotificationTrigger* trigger = [UNCalendarNotificationTrigger
       triggerWithDateMatchingComponents:date repeats:NO];
 
// Create the request object.
UNNotificationRequest* request = [UNNotificationRequest
       requestWithIdentifier:@"MorningAlarm" content:content trigger:trigger];
       
       
SWIFT

let content = UNMutableNotificationContent()
content.title = NSString.localizedUserNotificationString(forKey: "Wake up!", arguments: nil)
content.body = NSString.localizedUserNotificationString(forKey: "Rise and shine! It's morning time!",
                                                        arguments: nil)
 
// Configure the trigger for a 7am wakeup.
var dateInfo = DateComponents()
dateInfo.hour = 7
dateInfo.minute = 0
let trigger = UNCalendarNotificationTrigger(dateMatching: dateInfo, repeats: false)
 
// Create the request object.
let request = UNNotificationRequest(identifier: "MorningAlarm", content: content, trigger: trigger)
```

给 `UNNotificationRequest ` 对象提供一个标识符可以在通知调度之后用来识别通知。稍后可以使用标识符来查找未完成的请求或者在传递之前把它取消掉。更多关于调度和取消请求的信息，请看 [Scheduling Local Notifications for Delivery](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW5) 。

#### 给本地通知指定自定义动作

为了在界面上给本地通知显示自定义动作，需要在配置 [UNMutableNotificationContent](https://developer.apple.com/documentation/usernotifications/unmutablenotificationcontent) 对象时为 [categoryIdentifier](https://developer.apple.com/documentation/usernotifications/unnotificationcontent/1649866-categoryidentifier) 属性赋值一个已注册的分类的标识符。系统根据分类的信息来判定该显示哪个动作按钮，如果有，就把它显示在通知界面上。必须在调度通知请求前给这个属性赋值。

清单 3-2 演示了如何给本地通知指定分类标识符。在这个例子中，字符串 “TIMER_EXPIRED” 代表一个在启动时定义并且包含 2 个自定义动作的分类。注册这个分类的代码在清单 2-3 给出。

清单 3-2 给本地通知定义一个分类

```
OBJECTIVE-C

UNNotificationContent *content = [[UNNotificationContent alloc] init];
// Configure the content. . .
 
// Assign the category (and the associated actions).
content.categoryIdentifier = @"TIMER_EXPIRED";
 
// Create the request and schedule the notification.


SWIFT

let content = UNMutableNotificationContent()
// Configure the content. . .
 
// Assign the category (and the associated actions).
content.categoryIdentifier = "TIMER_EXPIRED"
 
// Create the request and schedule the notification.
```

关于如何使用分类注册自定义动作的信息，请看 [Configuring Categories and Actionable Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SupportingNotificationsinYourApp.html#//apple_ref/doc/uid/TP40008194-CH4-SW26) 。

#### 添加声音到通知内容中

如果需要在本地通知传递时播放一段音频，那么就给 `UNMutableNotificationContent ` 对象的 [sound](https://developer.apple.com/documentation/usernotifications/unmutablenotificationcontent/1649868-sound) 属性赋值。使用 [UNNotificationSound](https://developer.apple.com/documentation/usernotifications/unnotificationsound) 对象指定音频，该对象可以让你播放一段自定义音频或默认通知音频。自定义音频必须在播放前放置在本地。在应用主包中保存音频文件或者下载之后保存到应用的容器目录中的 `Library/Sounds` 子目录中。

要播放默认音频，需要创建音频文件并赋值给通知内容。例如：

```
OBJECTIVE-C

content.sound = [UNNotificationSound defaultSound];


SWIFT

content.sound = UNNotificationSound.default()
```

当指定自定义音频时，仅指定你想要播放的音频文件名。如果系统找到了与提供的文件名匹配的音频文件，就会在通知传递时播放该音频。如果系统没有找到合适的音频文件，就会播放默认的音频。

```
OBJECTIVE-C

content.sound = [UNNotificationSound soundNamed:@"MySound.aiff"];


SWIFT

content.sound = UNNotificationSound(named: "MySound.aiff")
```

有关支持的音频文件格式的信息，请看 [UNNotificationSound Class Reference](https://developer.apple.com/documentation/usernotifications/unnotificationsound)。

### 调度本地通知

要调度本地通知，需要创建 [UNNotificationRequest](https://developer.apple.com/documentation/usernotifications/unnotificationrequest) 对象并调用 `UNUserNotificationCenter` 的 [addNotificationRequest:withCompletionHandler:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649508-add) 方法。系统异步地调度本地通知，当调度完成或者有错误发生系统会调用完成处理块。清单 3-3 演示了如何调度本地通知。这个例子中的代码完善了在清单 3-1 中创建的通知调度例子。

清单 3-3 调度本地通知

```
OBJECTIVE-C

// Create the request object.
UNNotificationRequest* request = [UNNotificationRequest
       requestWithIdentifier:@"MorningAlarm" content:content trigger:trigger];
 
UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
   if (error != nil) {
       NSLog(@"%@", error.localizedDescription);
   }
}];


SWIFT

// Create the request object.
let request = UNNotificationRequest(identifier: "MorningAlarm", content: content, trigger: trigger)
 
// Schedule the request.
let center = UNUserNotificationCenter.current()
center.add(request) { (error : Error?) in
    if let theError = error {
        print(theError.localizedDescription)
    }
}
```

已调度的本地通知会保持活跃直到被系统清理调度或者被显式取消。通知在传递后系统会自动清理调度，除非通知的触发器配置为重复的。为了在一个独立的通知被传递前取消它，或者取消一个重复发生的通知，可以调用 `UNUserNotificationCenter` 的 [removePendingNotificationRequestsWithIdentifiers:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649517-removependingnotificationrequest) 方法。被取消通知的 [UNNotificationRequest](https://developer.apple.com/documentation/usernotifications/unnotificationrequest) 对象的 [identifier](https://developer.apple.com/documentation/usernotifications/unnotificationrequest/1649634-identifier) 属性必须有赋值。要取消所有未完成的本地通知(不管有没有请求标识符)，请调用 [removeAllPendingNotificationRequests](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649509-removeallpendingnotificationrequ) 方法。

### 响应通知的传递

当应用不在运行状态或者处于后台，系统会按照指定的交互方式自动地传递本地和远程通知。如果用户选中一个动作或者标准交互中的一个，系统就会把用户的选择通知应用，代码可以据此来执行额外的任务。如果应用正在前台运行，通知会直接传递到应用，然后你可以决定是安静地处理通知还是提醒用户。

为了响应通知的传递，必须实现 `UNUserNotificationCenter` 共享对象的代理方法。代理对象必须遵守 [UNUserNotificationCenterDelegate](https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate) 协议（通知中心用代理方法来传递通知信息到应用中）。如果通知包含自定义动作，代理是必须的。

>重要</br>
>必须在应用（或应用扩展）启动完之前给 `UNUserNotificationCenter` 对象的代理赋值，否则会妨碍应用正确地处理通知。

关于实现代理对象的额外信息，请看 [UNUserNotificationCenterDelegate Protocol Reference](https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate) 。

#### 当应用在前台时处理通知

如果通知在应用处于前台时到来，可以压制（silence）通知或者告知系统继续显示通知界面。系统在默认情况下会为前台应用压制通知，并把通知数据直接传递给应用，可以使用这些数据来直接更新应用界面。例如，如果新的运动比分数据到来，你只需要在你的界面中更新这些信息。

如果希望系统继续显示通知界面，需要给 `UNUserNotificationCenter` 提供代理对象并实现 [userNotificationCenter:willPresentNotification:withCompletionHandler:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate/1649518-usernotificationcenter) 方法，实现中依然处理通知数据。处理完之后执行提供的完成处理块并传入想让系统使用的传递选项（如果有的话）。如果没有指定任何选项，系统会压制通知。清单 3-4 给出了一个实现的例子，例子让系统播放一段音频，通知的负载会标识出要播放的音频。

清单 3-4 应用处于前台时播放音频

```
OBJECTIVE-C

- (void)userNotificationCenter:(UNUserNotificationCenter *)center
        willPresentNotification:(UNNotification *)notification
        withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler {
   // Update the app interface directly.
 
    // Play a sound.
   completionHandler(UNNotificationPresentationOptionSound);
}


SWIFT

func userNotificationCenter(_ center: UNUserNotificationCenter,
                            willPresent notification: UNNotification,
                            withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    // Update the app interface directly.
    
    // Play a sound.
    completionHandler(UNNotificationPresentationOptions.sound)
}
```

应用处于后台或者不在运行状态时，系统不会调用  `userNotificationCenter:willPresentNotification:withCompletionHandler:` 方法。在那些情况下，系统会根据通知中的信息来提示用户。你仍然可以使用 `UNUserNotificationCenter` 对象的 [getDeliveredNotificationsWithCompletionHandler:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649520-getdeliverednotificationswithcom) 方法来判定通知是否处于传递状态。

#### 响应自定义动作的选中

当用户在通知界面选中自定义动作，系统会把用户的选择结果通知应用。自定义按钮的响应会被打包到一个 [UNNotificationResponse](https://developer.apple.com/documentation/usernotifications/unnotificationresponse) 对象中并传递给应用的 [UNUserNotificationCenter](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter) 共享对象的代理。为了接收到响应，代理对象必须实现 [userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate/1649501-usernotificationcenter) 方法。方法的实现必须能够处理应用（或应用扩展）支持的所有自定义动作。

如果收到响应时应用（或应用扩展）不在运行状态，系统会在后台启动应用（或应用扩展）来处理响应。利用系统提供的后台时间来更新数据结构和应用界面，以此来反映用户的选择。不要用这部分时间来执行与自定义动作处理无关的任务。

清单 3-5 演示了一个计时应用（包含多个分类和自定义动作）响应处理方法的实现。实现利用动作和 [categoryIdentifier](https://developer.apple.com/documentation/usernotifications/unnotificationcontent/1649866-categoryidentifier) 属性来判定合适的路线。

清单 3-5 处理自定义的通知动作

```
OBJECTIVE-C

- (void)userNotificationCenter:(UNUserNotificationCenter *)center
           didReceiveNotificationResponse:(UNNotificationResponse *)response
           withCompletionHandler:(void (^)(void))completionHandler {
    if ([response.notification.request.content.categoryIdentifier isEqualToString:@"TIMER_EXPIRED"]) {
        // Handle the actions for the expired timer.
        if ([response.actionIdentifier isEqualToString:@"SNOOZE_ACTION"])
        {
            // Invalidate the old timer and create a new one. . .
        }
        else if ([response.actionIdentifier isEqualToString:@"STOP_ACTION"])
        {
            // Invalidate the timer. . .
        }
 
    }
 
    // Else handle actions for other notification types. . .
}


SWIFT

func userNotificationCenter(_ center: UNUserNotificationCenter,
                            didReceive response: UNNotificationResponse,
                            withCompletionHandler completionHandler: @escaping () -> Void) {
    if response.notification.request.content.categoryIdentifier == "TIMER_EXPIRED" {
        // Handle the actions for the expired timer.
        if response.actionIdentifier == "SNOOZE_ACTION" {
            // Invalidate the old timer and create a new one. . .
        }
        else if response.actionIdentifier == "STOP_ACTION" {
            // Invalidate the timer. . .
        }
    }
    
    // Else handle actions for other notification types. . .
}

```

#### 处理标准系统动作

在系统通知界面中，用户可以显式关闭通知界面或者启动应用而不是选中一个自定义动作。关闭界面涉及点击一个可用的按钮或者直接关闭界面，忽视通知或者轻轻弹开横幅通知不代表显式关闭。当系统动作被触发，用户通知中心把它们反馈到它的代理方法 [userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate/1649501-usernotificationcenter)。传递到方法中的响应对象包含以下动作标识符中的一个：

* [UNNotificationDismissActionIdentifier](https://developer.apple.com/documentation/usernotifications/unnotificationdismissactionidentifier) 让你知道用户没有选中自定义动作，而是显式关闭通知界面。
* [UNNotificationDefaultActionIdentifier](https://developer.apple.com/documentation/usernotifications/unnotificationdefaultactionidentifier) 让你知道用户没有选中自定义动作，而是启动应用。

以与处理其他动作同样的方式同样的方式来处理标准系统动作。清单 3-6 给出了检查这些特殊动作的 [userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate/1649501-usernotificationcenter) 方法的一个模版。

清单 3-6 处理标准系统动作

```
OBJECTIVE-C

- (void)userNotificationCenter:(UNUserNotificationCenter *)center
          didReceiveNotificationResponse:(UNNotificationResponse *)response
          withCompletionHandler:(void (^)(void))completionHandler {
   if ([response.actionIdentifier isEqualToString:UNNotificationDismissActionIdentifier]) {
       // The user dismissed the notification without taking action.
   }
   else if ([response.actionIdentifier isEqualToString:UNNotificationDefaultActionIdentifier]) {
       // The user launched the app.
   }
 
   // Else handle any custom actions. . .
}


SWIFT

func userNotificationCenter(_ center: UNUserNotificationCenter,
                            didReceive response: UNNotificationResponse,
                            withCompletionHandler completionHandler: @escaping () -> Void) {
    if response.actionIdentifier == UNNotificationDismissActionIdentifier {
        // The user dismissed the notification without taking action
    }
    else if response.actionIdentifier == UNNotificationDefaultActionIdentifier {
        // The user launched the app
    }
    
    // Else handle any custom actions. . .
}
```

## 配置远程通知支持

支持远程通知可以让你把最新的信息提供给应用，即使此时应用不处于运行状态。为了能够接收和处理远程通知，应用必须：

1. 启用远程通知
2. 向苹果`推送通知服务`（APNs）注册并接收应用特有的`设备令牌`（device token）
3. 把设备令牌发送到通知提供者服务器
4. 支持处理到来的远程通知

这一章节讲解了这些步骤，在应用中实现每一个步骤。关于提供者（开发者部署和管理的用来创建和发送通知请求到 APNs 的服务器），请看 [APNs Overview](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1)。

>注意</br>
>应用至少启动过一次，APNs 才能给未处于运行状态的应用传递远程通知。
>在 iOS 设备上，如果用户在多任务管理界面中强行退出了应用，那么应用不会再收到远程通知直到用户再次重启应用。

### 开启推送通知的能力

应用要处理远程通知，必须有正确的权限来与 APNs 对话。按照 Xcode 帮助描述[Enable push notifications](http://help.apple.com/xcode/mac/current/#/dev11b059073?sub=dev73a37248c) 使用 Xcode 工程的能力面板来给应用添加这个权限。

没有预期权限的应用在应用商店的审核流程中会被拒绝，在测试过程中，试图在没有正确授权的应用中向 APNs 注册会返回错误。

### 注册接收远程通知

每次应用启动必须向 APNs 注册，不同的平台使用的方法会有区别，但工作的流程都一样：

1. 应用请求向 APNs 注册
2. 成功注册后，APNs 将应用特有的设备令牌发送到设备
3. 系统通过调用应用的代理方法把设备令牌（原文中是 device，但按照上下文来理解，应该是 device token）传递到应用中
4. 应用把设备令牌发往与应用关联的提供者服务器

演示这些步骤的代码片段，请看 [Obtaining a Device Token in iOS and tvOS](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW4) 和 [Obtaining a Device Token in macOS](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW6)。

应用特有的设备令牌是全局唯一的并且标识一个应用和设备的组合。当在应用中收到来自 APNs 的设备令牌，需要打开一条到提供者的网络连接并把设备令牌连同其它任何相关的数据发送给提供者。提供者在向 APNs 发送远程通知请求时，必须附上设备令牌以及通知负载。更多相关内容，请看 [APNs Overview](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1)。

永远不要在应用中缓存设备令牌，而是在需要的时再从系统获取。APNs 会在发生某些事件时签发新的设备令牌，这个设备令牌肯定不一样，例如，当用户从备份中恢复设备时，当在新设备上安装应用时，当用户重装系统时。重新获取令牌而不是依赖于缓存，确保当前的设备令牌正是提供者需要用来与 APNs 通信的那个。当试图获取设备令牌时，如果设备令牌没有变，获取方法会马上返回。

>重要
>当设备令牌改变了，用户必须启动应用一次，APNs 才能再次把远程通知发送到设备

在 watchOS 上运行的应用不会显式注册远程通知，而是依赖配对的 iPhone 转发远程通知以显示在 watch 上，通知的转发会发生在（iPhone 处于锁定状态或者屏幕休眠并且 APPle Watch 在用户手腕上且已解锁）。

关于远程通知的数据格式和如何发送数据到 APNs 的信息，请看 [Communicating with APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW1)。

#### 在 iOS 和 tvOS 上获取设备令牌

在 iOS 和 tvOS 上，通过调用 `UIApplication` 的 [registerForRemoteNotifications](https://developer.apple.com/documentation/uikit/uiapplication/1623078-registerforremotenotifications) 方法来发起 APNs 注册。在启动时调用这个方法并把它当作正常启动流程的一部分。当应用首次调用这个方法时，应用对象会代你联系 APNs 并请求一个应用特有的设备令牌。然后系统会根据成功或失败异步调用以下代理方法：

* 设备令牌成功签发，系统调用 [application:didRegisterForRemoteNotificationsWithDeviceToken:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622958-application) 方法，实现这个方法来接收令牌并转发给提供者。
* 签发失败，系统调用 [application:didFailToRegisterForRemoteNotificationsWithError:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622962-application) 方法，实现这个方法来响应 APNs 注册错误。

>重要</br>
>APNs 设备令牌长度是可变的，不要硬编码。

成功向 APNs 注册之后，应用对象只会在设备令牌发生改变时与 APNs 联系，除此之外，调用 `registerForRemoteNotifications ` 方法会调用 `application:didRegisterForRemoteNotificationsWithDeviceToken:` 方法并立马返回现有令牌。

>注意</br>
>如果在应用运行的时候设备令牌发生改变，应用对象会再次调用 `application:didRegisterForRemoteNotificationsWithDeviceToken:` 代理方法通知你。

清单 4-1 演示了在 iOS 或者 tvOS 应用中如何获取设备令牌。应用代理把调用 `registerForRemoteNotifications` 方法当作常规启动时准备工作的一部分。在收到设备令牌时，`application:didRegisterForRemoteNotificationsWithDeviceToken:` 方法使用自定义的方法把设备令牌发送到关联的提供者，如果在注册过程中发生错误，应用会临时禁用任何与远程通知相关的功能，这些功能会在收到有效的设备令牌时重新启用。

清单 4-1 在 iOS 中注册远程通知

```
OBJECTIVE-C

- (void)applicationDidFinishLaunching:(UIApplication *)app {
    // Configure the user interactions first.
    [self configureUserInteractions];
 
   // Register for remote notifications.
    [[UIApplication sharedApplication] registerForRemoteNotifications];
}
 
// Handle remote notification registration.
- (void)application:(UIApplication *)app
        didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)devToken {
    // Forward the token to your provider, using a custom method.
    [self enableRemoteNotificationFeatures];
    [self forwardTokenToServer:devTokenBytes];
}
 
- (void)application:(UIApplication *)app
        didFailToRegisterForRemoteNotificationsWithError:(NSError *)err {
    // The token is not currently available.
    NSLog(@"Remote notification support is unavailable due to error: %@", err);
    [self disableRemoteNotificationFeatures];
}


SWIFT

func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    // Configure the user interactions first.
    self.configureUserInteractions()
    
    // Register with APNs
    UIApplication.shared.registerForRemoteNotifications()
}
 
// Handle remote notification registration.
func application(_ application: UIApplication,
                 didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data){
    // Forward the token to your provider, using a custom method.
    self.enableRemoteNotificationFeatures()
    self.forwardTokenToServer(token: deviceToken)
}
 
func application(_ application: UIApplication,
                 didFailToRegisterForRemoteNotificationsWithError error: Error) {
    // The token is not currently available.
    print("Remote notification support is unavailable due to error: \(error.localizedDescription)")
    self.disableRemoteNotificationFeatures()
}
```

如果网络（蜂窝或 Wi-Fi）不可用，[application:didRegisterForRemoteNotificationsWithDeviceToken:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622958-application) 和 [application:didFailToRegisterForRemoteNotificationsWithError:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622962-application) 方法都不会被调用。对于 Wi-Fi 连接来说，如果设备不能通过配置的端口连接到 APNs 就会出现这种情况，如果发生这种情况，用户可以移步到其它不阻塞期望端口的 Wi-Fi 网络。对于使用蜂窝通信的设备，用户可以一直等到蜂窝数据服务可用。

在 `application:didFailToRegisterForRemoteNotificationsWithError:` 方法的实现中，使用错误对象来禁用任何与远程通知相关的功能。因为无论如何通知不可能到来，所以最好优雅地降级并避免任何对促进远程通知有帮助的本地工作。之后如果远程通知可用，应用对象会通过调用 `application:didRegisterForRemoteNotificationsWithDeviceToken:` 代理方法来通知你。

#### 在 macOS 上获取设备令牌

在 macOS 中，通过调用 `NSApplication` 对象的 [registerForRemoteNotificationTypes:](https://developer.apple.com/documentation/appkit/nsapplication/1428476-registerforremotenotificationtyp) 方法来获取设备令牌。推荐在应用启动的时候调用这个方法，把它当作正常启动流程的一部分。应用首次调用这个方法时，应用对象会从 APNs 请求令牌。在这之后，应用对象仅仅会在
设备令牌发生改变时联系 APNs，除此之外，调用此方法会立即返回现有令牌。

应用对象会在获取设备令牌成功或失败时异步通知它的代理，可以在这些代理回调中处理设备令牌或任何出现的错误。必须实现以下方法来追踪注册的成功与否：

* 使用 [application:didRegisterForRemoteNotificationsWithDeviceToken:](https://developer.apple.com/documentation/appkit/nsapplicationdelegate/1428766-application) 来接收设备令牌并转发给提供者
* 使用 [application:didFailToRegisterForRemoteNotificationsWithError:](https://developer.apple.com/documentation/appkit/nsapplicationdelegate/1428554-application) 来响应错误

>注意</br>
>如果在应用运行的时候设备令牌发生改变，应用对象会再次调用合适的代理方法来通知你。

清单 4-2 演示了如何为 macOS 应用获取设备令牌。应用代理把调用 `registerForRemoteNotificationTypes:` 方法当作常规启动时准备工作的一部分，一并传入打算使用的交互类型。在收到设备令牌时，`application:didRegisterForRemoteNotificationsWithDeviceToken:` 方法会使用一个自定义方法把令牌转发给与应用关联的提供者。如果注册过程中发生错误，应用会临时禁用语远程通知关联的功能，这些功能会在收到有效的设备令牌时重新启用。

清单 4-2 在 macOS 上注册远程通知

```
OBJECTIVE-C

- (void)applicationDidFinishLaunching:(NSNotification *)notification {
    // Configure the user interactions first.
    [self configureUserInteractions];
 
    [NSApp registerForRemoteNotificationTypes:(NSRemoteNotificationTypeAlert | NSRemoteNotificationTypeSound)];
}
 
- (void)application:(NSApplication *)application
        didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    // Forward the token to your provider, using a custom method.
    [self forwardTokenToServer:deviceToken];
}
 
- (void)application:(NSApplication *)application
        didFailToRegisterForRemoteNotificationsWithError:(NSError *)error {
    NSLog(@"Remote notification support is unavailable due to error: %@", error);
    [self disableRemoteNotificationFeatures];
}


SWIFT

func applicationDidFinishLaunching(_ aNotification: Notification) {
    // Configure the user interactions first.
    self.configureUserInteractions()
    
    NSApplication.shared().registerForRemoteNotifications(matching: [.alert, .sound])
}
 
// Handle remote notification registration.
func application(_ application: NSApplication,
                 didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    // Forward the token to your provider, using a custom method.
    self.forwardTokenToServer(token: deviceToken)
}
 
func application(_ application: NSApplication,
                 didFailToRegisterForRemoteNotificationsWithError error: Error) {
    // The token is not currently available.
    print("Remote notification support is unavailable due to error: \(error.localizedDescription)")
}
```

### 处理远程通知

User Notifications 框架为 iOS、watchOS 和 tvOS 的应用提供了统一的 API 并支持大多数与本地和远程通知相关的任务。下面是一些可以用这个框架来执行的任务的例子：

* 如果应用在前台，可以直接接收通知并压制它
* 如果应用在后台或不在运行状态
	* 可以在用户选中了与通知关联的自定义动作时作出响应
	* 可以在用户关闭通知或者启动应用时作出响应

应用通过应用代理来接收远程通知的负载，当远程通知到来时，如果应用在后台，系统会按正常的用户交互方式来处理。在 iOS 和 tvOS 上，系统会把通知负载传递给应用代理的 [application:didReceiveRemoteNotification:fetchCompletionHandler:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623013-application) 方法。在 macOS 上，系统会把负载传递给应用代理的 [application:didReceiveRemoteNotification:](https://developer.apple.com/documentation/appkit/nsapplicationdelegate/1428430-application) 方法。可以使用这些方法检查负载和执行相关的任务。例如，当收到后台更新的远程通知时，你可能会为应用下载新内容。

有关如何使用 User Notifications 框架的方法来处理通知的信息，请看 [Responding to the Delivery of Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW14)。有关如何在应用代理中处理通知的信息，请看 [UIApplicationDelegate Protocol Reference](https://developer.apple.com/documentation/uikit/uiapplicationdelegate) 或者 [NSApplicationDelegate Protocol Reference](https://developer.apple.com/documentation/appkit/nsapplicationdelegate)。

## 修改和展示通知

可以使用应用扩展来修改通知的内容和展示。要在远程通知传递之前修改它的内容，使用 `通知服务应用扩展`。要修改通知在屏幕上的显示方式，使用 `通知内容应用扩展`。

### 修改远程通知的负载

在远程通知传递给用户前，使用通知服务应用扩展来修改负载。远程通知来自服务器，服务器掌控通知的配置和内容。服务扩展可以让应用在数据呈现给用户前修改服务器提供的负载数据。使用服务扩展可以实现以下功能：

* 解密经过加密的数据
* 下载图片以及其它的媒体文件并把它当作附件加入到通知中
* 修改通知的主体或标题文本
* 给通知增加线程标识符或者修改通知的 [userInfo](https://developer.apple.com/documentation/usernotifications/unnotificationcontent/1649869-userinfo) 字典的内容

#### 添加通知服务应用扩展到应用中

1. 在 Xcode 选择 New > Target 添加一个 target 到工程中
2. 在 iOS > Application Extension 选项中，选择 Notification Service Extension target
3. 点击 Next
4. 指定应用扩展名和其它的详细内容
5. 点击 Finish

Xcode 会添加一个预先配置好的 target 到工程中。

由 Xcode 提供的默认通知服务扩展 target 包含一个可以修改的 `UNNotificationServiceExtension ` 子类。使用 [didReceiveNotificationRequest:withContentHandler:](https://developer.apple.com/documentation/usernotifications/unnotificationserviceextension/1648229-didreceivenotificationrequest) 方法创建和配置一个新的 [UNMutableNotificationContent](https://developer.apple.com/documentation/usernotifications/unmutablenotificationcontent) 对象，可根据需要修改这个新内容对象，替换一些或所有初始内容值。在完成修改之后，调用完成处理块并传入这个新内容对象。系统会把新内容整合到通知中并传递给用户。

系统仅提供有限的时间来修改和调用完成处理块，所以你应该快速完成所有任务。如果在 `didReceiveNotificationRequest:withContentHandler:` 方法中花费了太多时间导致没有调用完成处理块，系统会调用 [serviceExtensionTimeWillExpire](https://developer.apple.com/documentation/usernotifications/unnotificationserviceextension/1648227-serviceextensiontimewillexpire) 方法给你一个最后的机会来完成修改。如果没有及时调用完成处理块，系统会显示原来的通知内容。

由服务器发送的远程通知应该精心处理，以便支持通知服务应用扩展去修改。没有处理过的通知会直接传递给用户。当创建远程通知的负载时，服务器应该做以下事情：

* 包含名为 'mutable-content'，值为 1 的键值对
* 包含 `alert` 字典（字典中有 `title` 和 `body` 两个子键）

更多有关实现通知服务应用扩展的方法的信息，请看 [UNNotificationServiceExtension Class Reference](https://developer.apple.com/documentation/usernotifications/unnotificationserviceextension)。有关配置远程通知负载的信息，请看 [Creating the Remote Notification Payload](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW1)。

### 在 iOS 上使用自定义界面展示通知

使用 `通知内容应用扩展` 来为通知显示自定义的用户界面。使用这类扩展来嵌入自定义内容或者使用与默认界面不同的布局。例如，你可能会使用这类扩展来在通知中显示图片或媒体文件。

通知内容应用扩展支持显示与指定分类关联的本地和远程通知。使用 `UNNotificationContent` 对象的 [categoryIdentifier](https://developer.apple.com/documentation/usernotifications/unnotificationcontent/1649866-categoryidentifier) 属性给本地通知指定分类。对于远程通知，服务器会给负载的 `aps` 字典中的名为 `category` 键一个合适的值。当该分类的通知到来时，系统会从扩展中加载视图控制器并在系统界面中嵌入你的内容。在视图控制器显示到屏幕之前，使用通知内容来配置它。

#### 给 iOS 应用添加通知内容应用扩展

1. 在 Xcode 中选择 New > Target 来添加一个新 target 到工程中
2. 在 iOS > Application Extension 选项中，选择 Notification Content Extension target
3. 点击 Next
4. 指定扩展名和其它详细内容
5. 点击 Finish

Xcode 会添加一个预先配置好的 target 到工程中。

新建的通知内容应用扩展 target 是为了显示与单一分类关联的通知而配置的。必须修改 target 来指定每个扩展想要支持的分类。使用 target 的 `Info.plist` 文件中 `UNNotificationExtensionCategory ` 键来指定分类。把这个值设置成在应用启动时注册的 `UNNotificationCategory ` 对象的 [identifier](https://developer.apple.com/documentation/usernotifications/unnotificationcategory/1649276-identifier) 属性的值。 

#### 在应用扩展中支持多个通知分类

1. 选中通知内容扩展工程的 `Info.plist` 文件
2. 展开 `NSExtension` 字典查看与扩展相关的键
3. 展开 `NSExtensionAttributes` 字典
4. 把 `UNNotificationExtensionCategory` 键的值的类型修改为 Array
5. 为扩展处理的通知分类添加一个条目

在 iOS 的应用包中可能包含多个通知内容应用扩展。系统希望一个给定的分类仅由一个扩展来支持，所以必须给每个扩展的 `UNNotificationExtensionCategory ` 键配置不同的一组值。

更多有关实现通知内容应用扩展的信息，请看 [UNNotificationContentExtension Protocol Reference](https://developer.apple.com/documentation/usernotificationsui/unnotificationcontentextension)。

### 在 watchOS 上使用自定义界面展示通知

WatchKit 框架支持使用自定义界面展示通知。当通知抵达应用时，一个 WatchKit 扩展可能包含一个或更多的通知界面控制器来显示通知。可以使用这些界面控制器来展示通知的内容。有关如何实现通知界面控制器的信息，请看 [App Programming Guide for watchOS](https://developer.apple.com/library/content/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html#//apple_ref/doc/uid/TP40014969)。

# 苹果推送通知服务

## 苹果推送通知服务概览

*Apple Push Notification service*(苹果推送通知服务，简称 APNs)是远程通知功能的中枢。这是一个健壮的、安全的、高效的服务，用来帮助开发者传送信息到 iOS（间接传给 watchOS）、tvOS 和 macOS 设备。

当应用在用户的设备上初次启动时，系统会自动为应用建立一条到 APNs 的可信任的、加密的 IP 长连接。这条连接允许应用执行准备工作来让它能够接收通知，正如 [Configuring Remote Notification Support](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1) 描述的那样。

发送通知的连接的另一半是提供者服务器与 APNs 之间安全的长连接通道。这需要在线上 [开发者账户](https://developer.apple.com/account/) 中配置和使用苹果提供的密码证书。*提供者*是开发者部署和管理的服务器，经过配置之后用来和 APNs 一起工作。图 6-1 展示了一个远程通知的传递路径。

图 6-1 从提供者传递一条远程通知到应用

![从提供者传递一条远程通知到应用](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/remote_notif_simple_2x.png)

在提供者上和应用中做好推送通知的准备工作后，提供者可以向 APNs 发出通知请求，APNs 再把相应的通知负载转达给每一台目标设备。当系统收到通知时，系统会把负载传递给设备中合适的应用并管理与用户的交互。

如果通知到来时设备已开机但应用没有在运行状态，系统仍然可以显示通知。如果 APNs 发送通知时设备已关机，APNs 会保留通知并在稍后重发（详细内容，请看 [Quality of Service, Store-and-Forward, and Coalesced Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW5)）。

### 提供者的职责

为了参与 APNs，提供者有以下职责：

* 通过 APNs 接收全局唯一的、应用特有的设备令牌，接收来自应用实例的其它相关数据。这使得提供者清楚每一个运行的应用实例。
* 根据通知系统的设计来判定何时需要往设备发送通知。
* 创建并发送通知请求（每一个请求都包含通知负载和传递的信息）到 APNs，然后 APNs 代你把相应的通知传递给预期设备。

提供者发送的每一个远程通知请求必须：
1. 构造一个包含通知负载的 JSON 字典，如 [Creating the Remote Notification Payload](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW1) 所描述。
2. 把负载、全局唯一设备令牌和其它递送信息添加到一个 HTTP/2 请求中。有关设备令牌的信息，请看 [APNs-to-Device Connection Trust and Device Tokens](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW13)。有关 HTTP/2 请求的格式的信息和来自 APNs 的可能响应和错误，请看 [Communicating with APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW1)。
3. 通过一条安全的长连接通道发送 HTTP/2 请求到 APNs，请求包含令牌形式的密码凭证或者证书。建立安全通道在 [Security Architecture](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW9) 有讲解。

### 使用多个提供者

图 6-2 描述了 APNs 为应用运行启用的虚拟网络的分类。为了处理加载的通知，通常会部署多个提供者（每一个都有一条到 APNs 的安全长连接）。然后，提供者就可以发送请求到目标设备（提供者拥有的有效设备令牌对应的设备）。

图 6-2 从多个提供者推送远程通知到多个设备
![从多个提供者推送远程通知到多个设备](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/remote_notif_multiple_2x.png)

### 服务的质量、保存和转发、合并通知

苹果推送通知服务包含一个服务质量（Qulity of Service 简称 QoS）组件，该组件有保存和转发的功能。如果 APNs 试图发通知而此时目标设备处于离线状态，APNs 会保存通知一段有限时间并在设备上线时传递出去。这组件仅为一台设备的一个应用保留最近的通知。如果设备不在线，那么往这台设备发送通知请求会导致先前的请求被丢弃。如果设备长时间不在线，那么所有在 APNs 中保存的通知都会被丢弃。

要想允许合并类似的通知，你可以在通知的请求中包含 *collapse* 标识符。通常，当设备在线时，往 APNs 发送的通知请求都会传递到设备上。然而，当 `apns-collapse-id` 键出现在 HTTP/2 请求头中时，APNs 会合并该键的值相同的请求。例如，一个新闻服务发送了两次同样的头条，可以给这两个请求使用同样的合并标识符。然后，APNs 会把这两个请求合并到一个通知中发送给设备。关于 `apns-collapse-id` 键的细节，请看 [表 8-2](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW13)。

### 安全体系结构

APNs 使用信任的 2 个层级(`connection trust` 连接信任和 `device token trust` 设备令牌信任)来加强端到端的密码验证和认证。

*Connection trust* 在提供者和 APNs 之间、APNs 和设备之间工作。
	
**提供者到 APNs 的连接信任** 建立确定性（提供者和 APNs 之间的连接只可能为授权的提供者打开，授权的提供者由与苹果签订有推送通知传递协议的公司拥有）。必须确保在提供者服务器与 APNs 之间存在连接信任，如本章节所描述。
**APNs 到设备的连接信任** 确保只有授权的设备才能连接到 APNs。APNs 自动加强与每台设备的连接信任以确保设备的合法性。

与 APNs 通信的提供者，必须采用有效的认证秘钥证书（基于令牌的连接信任）或 SSL 证书（基于证书的连接信任）。可在线上 [开发者账户](https://developer.apple.com/account/) 获得这两个证书，正如 Xcode 帮助中心 [Configure push notifications](http://help.apple.com/xcode/mac/current/#/dev11b059073) 所描述的那样。在两个证书类型中选择哪一个，请看 [Provider-to-APNs Connection Trust](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW4)。无论选择哪种证书，提供者连接信任是提供者发送推送通知请求到 APNs 的先决条件。

*Device token trust* 在每条远程通知的端到端工作。它确保通知从正确的起点(提供者)路由到正确的终点（设备）。

设备令牌是一个不透明的 NSData 实例，它包含一个苹果为特定设备上的特定应用指定的唯一标识符。只有 APNs 可以解码和读取设备令牌的内容。每个应用在向 APNs 注册时会收到唯一的设备令牌，然后必须把令牌转发给提供者，如 [Configuring Remote Notification Support](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1) 所描述。提供者必须在每条指向关联设备的推送通知请求中包含设备令牌，APNs 使用设备令牌来确保通知只传递给预期的 `应用-设备` 唯一组合。

APNs 可能会发布新的设备令牌，由于以下原因：

* 用户在新设备上安装应用
* 用户从备份中还原设备
* 用户重装系统
* 其它系统定义事件

因此，应用必须在启动时请求设备令牌，如 [APNs-to-Device Connection Trust and Device Tokens](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW13) 所描述的那样。代码示例请看 [Registering to Receive Remote Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW3)。

>重要</br>
>为了保护用户的隐私，不要使用设备令牌来识别用户设备。

#### 提供者到 APNs 的连接信任

提供者和 APNs 之间协商连接信任的方案有两种：

**基于令牌的连接信任**：使用基于 HTTP／2 的 API 的提供者可以使用 *JSON web tokens(JWT)* 来为与 APNs 的连接提供验证凭证。在这个方案中，公钥提供给苹果保管，私钥由自己保管。然后，提供者使用私钥来生成和签名 *JWT 提供者授权令牌*。每一个推送消息请求必须包含提供者授权令牌。

可以使用一个单独的基于令牌的连接（在提供者和 APNs 之间）来给所有应用（线上[开发者账户](https://developer.apple.com/account/) 中罗列的 bundle ID 对应的应用）发推送通知请求。

提供者发出的每一个推送通知请求都会收到一个来自 APNs 的 HTTP／2 响应（成功或失败的细节）。

**基于证书的提供者连接信任**：提供者可以选择采用唯一的*提供者证书(provider certificate)和私有密钥(private cryptographic key)*。提供者证书由苹果提供（当你在线上 [开发者账户](https://developer.apple.com/account/) 创建推送服务时），它标识一个主题，也是你应用的 bundle ID。

可以在提供者和 APNs 之间使用基于证书的连接来发送通知请求到应用（在线上 [开发者账户](https://developer.apple.com/account/) 配置证书的时候指定的应用）。

>重要</br>
>想要与 APNs 建立基于 HTTP／2 的 TLS 会话，必须确保提供者上已经安装好 GeoTrust Global CA 根证书。如果提供者运行在 macOS 上，这个根证书默认放在钥匙串中，其它操作系统可能需要显式安装。可以从 [GeoTrust Root Certificates website](https://www.geotrust.com/resources/root-certificates/) 下载证书，这是传送门 [direct link to the certificate](https://www.geotrust.com/resources/root_certificates/certificates/GeoTrust_Global_CA.pem)。
>然而，如果还使用遗留的二进制接口，必须确保每个提供者都有一个 Entrust Certification Authority (2048) 根证书，可向 [Entrust SSL Certificates website](https://www.entrust.com/ssl-certificates/) 购买。

##### 基于令牌的提供者到 APNs 的信任

基于令牌的提供者信任采用 "Apple Push Notification Authentication Key (Sandbox & Production)" 这类证书。按照 Xcode 帮助文档 “[Generate a universal provider token signing key](http://help.apple.com/xcode/mac/current/#/dev11b059073?sub=dev1eb5dfe65)” 使用线上 [开发者账户](https://developer.apple.com/account/) 配置和获得证书。这类证书有以下特点：

* 证书是有效的，可用来为与账户关联的每个应用发送推送通知请求，也可以为应用建立到 Appple Watch Complications(不好翻译 [wikipedia complication](https://en.wikipedia.org/wiki/Complication_(horology)) 的连接和获取 voice-over-Internet Protocol（VoIP）状态的通知。即使这些应用处于后台，APNs 依然会传递通知。详细请看 [APNs Provider Certificates](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW5)，还有 [Energy Efficiency Guide for iOS Apps](https://developer.apple.com/library/content/documentation/Performance/Conceptual/EnergyGuide-iOS/index.html#//apple_ref/doc/uid/TP40015243) 中的 [Voice Over IP (VoIP) Best Practices](https://developer.apple.com/library/content/documentation/Performance/Conceptual/EnergyGuide-iOS/OptimizeVoIP.html#//apple_ref/doc/uid/TP40015243-CH30)。
* 当通过 JWT 基于令牌的 APNs 连接发送推送通知请求时，必须包含提供者认证令牌
* APNs 认证秘钥证书永不过期，但可以使用线上 [开发者账户](https://developer.apple.com/account/) 永久废弃它，一旦废弃，就不能再使用了。

图 6-3 阐述使用基于 HTTP/2 的提供者 API 来建立信任和使用 JWT 提供者认证令牌发送通知。

图 6-3 建立和使用基于令牌的提供者连接信任
![建立和使用基于令牌的提供者连接信任](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/service_provider_ct_2x.png)

如图 6-3 所示，基于令牌的提供者信任工作流程如下：

1. 提供者使用 TLS 请求与 APNs 建立安全连接，如图中注 “TLS initiation” 的箭头所示
2. APNs 返回 APNs 证书（提供者稍后对其进行校验）给提供者，如图中第二个标注 “APNs certificate” 的箭头所示。此时，连接信任已经建立，提供者可以发送基于令牌的远程通知请求到 APNs。
3. 每一个提供者发送的通知请求必须随 JWT 授权令牌一起发送，如图中标注 “Notification push” 的箭头所示
4. APNs 回应每一条通知，如图中标注 “HTTP/2 response” 的箭头所示。在这一步中提供者可能收到的响应的相关细节，请看 [HTTP/2 Response from APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW2)。

##### 基于证书的提供者到 APNs 的信任

基于证书的提供者连接是有效的，可用来传递通知到指定应用（由提供者证书中的主题指定，主题为应用的 bundle ID）。提供者证书必须事先创建好，如 Xcode help 中 [Generate a universal APNs client SSL certificate](http://help.apple.com/xcode/mac/current/#/dev11b059073?sub=dev2178d70ae) 描述。取决于你如何配置和供应证书，受信任的连接可用来传递通知到其它与应用关联的事物，包括 Apple Watch complications 和 VoIP 状态通知。APNs 甚至会在它们处于后台时传递通知。详见 [Communicating with APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW1)，还有 [Energy Efficiency Guide for iOS Apps](https://developer.apple.com/library/content/documentation/Performance/Conceptual/EnergyGuide-iOS/index.html#//apple_ref/doc/uid/TP40015243) 中的 [Voice Over IP (VoIP) Best Practices](https://developer.apple.com/library/content/documentation/Performance/Conceptual/EnergyGuide-iOS/OptimizeVoIP.html#//apple_ref/doc/uid/TP40015243-CH30)。

对于基于证书的信任，APNs 会维护一张证书废弃清单，如果提供者的证书在废弃清单上，APNs 可以撤回提供者信任，也就是说，APNs 会拒绝 TLS 初始化连接。

图 6-4 阐述使用苹果签发的 SSL 证书来建立提供者和 APNs 之间的连接。不像 图 6-3，这张图没有展示通知推送部分，而是停在 TLS 连接建立这一步。在基于证书的信任方案中，通知请求没有经过认证但会使用随通知一同发出的设备令牌来验证。

图 6-4 建立基于证书的提供者连接信任
![建立基于证书的提供者连接信任](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/service_provider_ct_certificate_2x.png)

如图 6-4 所示，基于证书的提供者到 APNs 的信任工作流程如下：

1. 提供者使用 TLS 请求与 APNs 建立安全连接，如图中标注为 “TLS initiation” 的箭头所示
2. APNs 把 APNs 的证书（提供者稍后会进行验证）发给提供者，如图中标注为 “APNs certificate” 的箭头所示
3. 提供者必须把苹果提供的证书发给 APNs。这个证书已事先通过线上 [开发者账户](https://developer.apple.com/account/) 获得，如 Xcode Help [Generate a universal APNs client SSL certificate](http://help.apple.com/xcode/mac/current/#/dev11b059073?sub=dev2178d70ae) 中描述。如图中标注为 “Provider certificate” 的箭头所示
4. 紧接着 APNs 会验证提供者的证书，从而确认连接请求来自合法的提供者，然后建立 TLS 连接。此时，连接信任已经建立，提供者可以发送基于证书的远程通知请求到 APNs。

#### APNs 到设备的连接信任和设备令牌

APNs 和每台设备间的信任是自动建立的，不需要应用的参与。

每台设备都有一个密码证书和一个私钥，在设备首次激活的时候由操作系统提供并存放在设备的钥匙串中。在激活的过程中，APNs 基于证书和秘钥认证和验证到设备的连接，如图 6-5 所示。

图 6-5 建立设备和 APNs 的连接信任
![建立设备和 APNs 的连接信任](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/service_device_ct_2x.png)

如图 6-5，APNs 和设备的信任工作流程如下：
1. 信任协商从设备初始化一条到 APNs 的 TLS 连接开始，如图中顶部箭头
2. APNs 发送 APNs 证书给设备
3. 操作系统验证 APNs 证书，然后把设备的证书发给 APNs，如图中标注为 “Device certificate” 的箭头所示
4. 最后，APNs 验证设备的证书，然后建立信任，如图中底部箭头所示

有了设备和 APNs 间建立的 TLS 连接，设备上的应用就可以向 APNs 注册以获取应用特有的设备令牌，以便发送远程通知。详情和代码示例，请看 [Configuring Remote Notification Support](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1) 中的 [Registering to Receive Remote Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW3)。

收到设备令牌后，应用必须连接到与之关联的提供者并把设备令牌转发给它。这一步是必须的，因为之后提供者往 APNs 发送通知请求必须包含设备令牌以指定目标设备。转发令牌的代码也在 [Registering to Receive Remote Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW3) 中。

无论用户是否首次激活设备或者 APNs 是否已经发布了新的设备令牌，流程都与图 6-6 所示相类似。

图 6-6 管理设备令牌
![管理设备令牌](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/token_generation_2x.png)

获取和处理应用特有的设备令牌工作流程如下：

1. 应用向 APNs 注册远程通知，如图中顶部箭头所示。如果应用已经注册并且设备令牌没变，系统会立即返回现有设备令牌给应用，流程跳到第 4 步；
2. 当需要新的设备令牌时，APNs 使用设备证书包含的信息来生成一个新的，然后使用令牌秘钥来加密令牌并返回给设备，如图中中间向右的箭头所示；
3. 系统通过调用 [application:didRegisterForRemoteNotificationsWithDeviceToken:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622958-application) 代理方法把设备令牌传到应用中；
4. 当收到令牌时，应用必须在代理方法中把令牌转发给提供者（以二进制或十六进制格式），没有这个令牌提供者无法发送通知到设备。详见 [Configuring Remote Notification Support](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1) 中的 [Registering to Receive Remote Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW3)。

>重要</br>
>APNs 的设备令牌是可变长度的，不要硬编码。

当提供者往 APNs 发送推送通知请求时，会包含设备令牌（唯一标识应用与设备的组合）。这一步如图 6-7 中提供者与 APNs 之间的 “Token， Payload” 箭头所示。APNs 解密令牌以确保请求的合法性及判定目标设备。如果 APNs 判定发送者和接收者都是合法的，它就会把通知发到标识设备。

图 6-7 从提供者到设备的远程通知路线
![从提供者到设备的远程通知路线](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/token_trust_2x.png)

设备收到通知之后（并且在图 6-7 的最后一步之后），系统就把远程通知转发给应用。

### 配置程序

iOS 应用商店、tvOS 应用商店和 macOS 应用商店上的应用都可以使用 APNs，包括企业应用。为了使用 APNs，应用必须进行配置和代码签名。如果你只是团队中的一员，这些配置步骤的大部分只能由团队代理人或者管理员来执行。

关于如何在 Xcode 和线上 [开发者账户](https://developer.apple.com/account/) 中配置推送通知的支持，请看 Xcode Help [Configure push notifications](http://help.apple.com/xcode/mac/current/#/dev11b059073)。

## 创建远程通知负载

每一条提供者发往 APNs 的通知都包含一个负载。负载包含想要发给应用的自定义数据和系统通知用户的方式。把负载做成一个 JSON 字典并把它作为 HTTP/2 消息的主体内容发送出去。负载最大尺寸取决于所发送的通知：

* 常规的远程通知，最大尺寸是 4KB（4096 字节）
* VoIP 通知，最大尺寸是 5KB（5120 字节）

>注意</br>
>如果你使用遗留的 APNs 二进制接口来发送通知，而不是 HTTP/2 请求，那么负载的最大尺寸是 2KB（2048 字节）

APNs 会拒绝超过最大允许尺寸的通知负载。

因为通知的传递并没有保障，所以永远不要在负载中加入敏感数据以及可以通过其它方式获取的数据。相反，应该使用通知来提醒用户有新的信息或把它当作信号（应用有数据等着它）。例如，一个邮件应用使用远程通知来显示应用角标或弹框提醒用户指定的账户有新邮件，而不是直接在通知中发送邮件的内容。当收到通知时，应用应当连接到邮件服务器拉取邮件消息。

### 创建 JSON 字典

下面的例子说明了 JSON 字典的结构以及可以在通知负载中使用的键。该负载中最重要的部分是 `aps` 字典，其中包含苹果定义的键（用来限定系统在收到通知时如何提醒用户）。例子中还包含名为 “acme” 的键，代表虚构应用的自定义数据。例子中包含空格和换行是为了可读性，但是在实践中应该删除空格和换行符以减小负载的尺寸。

**例 1.** 下面的负载包含一个 `aps` 字典，字典中有一个简单的提示消息。`acme` 键包含一个应用特有的数据的数组。

```
{
    "aps" : { "alert" : "Message received from Bob" },
    "acme2" : [ "bang",  "whiz" ]
}
```

**例 2.** 下面的负载要求系统在显示消息时展示一个关闭按钮和一个动作按钮。`title` 和 `body` 键提供消息的内容，“PLAY” 字符串用来从应用的 `Localizable.strings` 文件中获取本地化字符串，获取到的字符串结果用来当作消息的动作按钮标题。这个负载还要求系统在应用图标上显示角标，数字为 5。

```
{
    "aps" : {
        "alert" : {
            "title" : "Game Request",
            "body" : "Bob wants to play poker",
            "action-loc-key" : "PLAY"
        },
        "badge" : 5
    },
    "acme1" : "bar",
    "acme2" : [ "bang",  "whiz" ]
}
```

**例 3.** 下面的负载规定设备应该显示一个弹框消息，播放声音和显示应用角标
```
{
    "aps" : {
        "alert" : "You got your emails.",
        "badge" : 9,
        "sound" : "bingbong.aiff"
    },
    "acme1" : "bar",
    "acme2" : 42
}
```

**例 4.** 下面的负载使用 `loc-key` 来指定应用的 `Localizable.strings ` 文件中的一个本地化字符串，该字符串被当作弹框的消息来显示，`loc-args` 包含的值可以在该字符串显示之前组合到该字符中。负载也指定了一个要随消息播放的自定义声音。

```
{
    "aps" : {
        "alert" : {
            "loc-key" : "GAME_PLAY_REQUEST_FORMAT",
            "loc-args" : [ "Jenna", "Frank"]
        },
        "sound" : "chime.aiff"
    },
    "acme" : "foo"
}
```

完整的苹果指定键（通知负载中可以包含的键）清单，请看 [Payload Key Reference](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/PayloadKeyReference.html#//apple_ref/doc/uid/TP40008194-CH17-SW1)。

### 配置后台更新通知

后台更新通知通过这样的方式（周期性的唤醒应用以便在后台刷新数据）来提升用户体验。应用长时间不运行，应用的数据可能已经过时，当用户再次启动应用，过时的数据必须要换掉（这会导致应用的使用被延迟）。后台更新通知可以提醒用户或者安静地出现。

>重要</br>
>后台更新通知并非一种让应用长期在后台处于唤醒状态的方式（仅限于快速刷新的操作），也不是高优先级的更新。APNs 把它当作一种低优先级的通知，并且如果发送的数量过多可能会被节流。实际的上限是动态的并且根据情况会有所变化，但是不要试图在一小时内发送过多的通知。

需要支持后台更新通知，请确保负载的 `aps` 字典中包含值为 1 的 `content-available` 键。如果后台更新中有用户可见的更新，可以在 `aps` 字典中适当地使用 `alert`、 `sound` 和 `badge` 等键。

当后台更新通知传递到用户设备，iOS 会在后台唤醒应用并给予多达 30s 的时间来运行。在 iOS 中，系统通过调用应用代理方法 [application:didReceiveRemoteNotification:fetchCompletionHandler:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623013-application) 来传递后台更新通知。使用该方法来初始化需要的操作以拉取新数据。在后台处理远程通知需要在应用中添加合适的后台模式(background modes)。

#### 配置应用来处理后台更新通知

1. 在工程导航器中选择工程
2. 在编辑器中选择 iOS 应用 target
3. 选中 Capabilities 选项
4. 启用 Background Modes capability
5. 启用 Remote notification background mode

![EnableRemoteNotificationBackgroundMode](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/remote_notification_mode_2x.png)

清单 7-1 给出了后台更新通知的 JSON 负载的一个例子

清单 7-1 配置后台更新通知
```
{
    "aps" : {
        "content-available" : 1
    },
    "acme1" : "bar",
    "acme2" : 42
}
```

关于如何处理远程通知的信息，请看 [Handling Remote Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW2)。

### 给远程通知指定自定义动作

要在远程通知中显示自定义动作，请在通知负载的 `aps` 字典中包含 `category` 键。在启动的时候，应用可以注册包含自定义动作的分类。如果通知包含 `category` 键，系统在横幅或弹框界面中会把分类的动作显示成按钮。当用户选中按钮，系统会通知应用以便执行关联的任务。配置了分类的通知必须也要配置显示弹框。

清单 7-2 演示了显示弹框和包括自定义动作的分类的通知负载。 “NEW_MESSAGE_CATEGORY” 字符串与应用的已注册分类名对应。在这种情况下，分类包含自定义动作以响应消息。

清单 7-2 在负载中包含分类
```
{
   "aps" : {
      "category" : "NEW_MESSAGE_CATEGORY"
      "alert" : {
         "body" : "Acme message received from Johnny Appleseed",
      },
      "badge" : 3,
      "sound" : “chime.aiff"
   },
   "acme-account" : "jane.appleseed@apple.com",
   "acme-message" : "message123456"
}
```

关于如何注册应用支持的分类和自定义动作的信息，请看 [Configuring Categories and Actionable Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SupportingNotificationsinYourApp.html#//apple_ref/doc/uid/TP40008194-CH4-SW26)。关于处理应用中自定义动作的选择，请看 [Responding to the Selection of a Custom Action](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW2)。

### 本地化远程通知的内容

有两种本地化远程通知内容的方式：

* 从提供者服务器提供本地化内容
* 在应用包中储存本地化消息字符串

两种本地化方式都有各自的优点和缺点，应该选择最适合自己需求的技术。从服务器提供本地化内容可以让你指定你想要的任何文本，但这需要你动态地发现和追踪用户的当前语言偏好设置以及潜在地翻译内容。在应用包中储存本地化字符串的方式相对简单些，但需要事先定义好所有的通知消息并放入应用的 `Localizable.strings` 资源文件中。

#### 为服务器提供用户语言偏好

如果由服务器来处理通知消息的本地化，应用应该向服务器传达用户的语言偏好设置。用户在设备上设置语言偏好，应用可以使用 `NSLocale` 类的 [preferredLanguage](https://developer.apple.com/documentation/foundation/nslocale/1415614-preferredlanguages) 属性来获取这些偏好设置。

清单 7-3 说明了获取当前选中的语言并传送给提供者服务器的技术。这例子获取用户的第一偏好语言并编码为 UTF8 字符串，然后使用自定义方法把字符串发给提供者。你可能会考虑发送从 [preferredLanguage](https://developer.apple.com/documentation/foundation/nslocale/1415614-preferredlanguages) 属性获取的少数第一语言以免用户的第一语言不是你支持的那种。如果不支持用户的偏好语言，可以考虑把像英语或者西班牙语这类广泛使用的语言当作备选方案。

清单 7-3 获取当前支持的语言并发送给提供者
```
NSString *preferredLang = [[NSLocale preferredLanguages] objectAtIndex:0];
const char *langStr = [preferredLang UTF8String];
[self sendProviderCurrentLanguage:langStr]; // custom method
}
```
因为用户可能会修改偏好语言设置，所以应用应当观察 [NSCurrentLocaleDidChangeNotification](https://developer.apple.com/documentation/foundation/nslocale/1418141-currentlocaledidchangenotificati) 通知。使用该通知来发送语言相关的变化到服务器。

#### 在应用包中储存本地化内容

如果通知使用一组一致的消息，你可以在应用包中保存消息文本的本地化版本并使用负载中的 `loc-key` 和 `loc-args` 键来指定要显示的消息。`loc-key` 和 `loc-args` 键定义通知的消息内容。当通知呈现，本地系统会在 `Localizable.strings` 文件中查找与 `loc-key` 键的值相匹配键字符串，然后使用来自字符串文件的相应值作为消息文本的基点，并使用 `loc-args` 键指定的字符串替换占位值（也可以使用 `title-loc-key` 和 `title-loc-args` 键来为通知指定标题）。

为了说明如何使用这些键，细想一个游戏应用的例子，该应用在用户受邀时发送通知。因为邀请的文本不会改变，所以文本被包含在应用的 `Localizable.strings` 文件中，使用以下条目：

`"GAME_PLAY_REQUEST_FORMAT" = "%@ and %@ have invited you to play Monopoly";`

当提供者服务器想要发送一个有关游戏的通知请求时，它会使用 `loc-key` 和  `loc-args` 键来创建负载。它把 `loc-key` 的值设置为 `GAME_PLAY_REQUEST_FORMAT ` 字符串，把 `loc-args` 值设置为参与者的名字，以此来初始化一个玩游戏的请求。例如，初始化两个用户名为 “Jenna” 和  “Frank” 的请求，提供者服务器会发送包含以下内容的负载：

```
{
    "aps" : {
        "alert" : {
            "loc-key" : "GAME_PLAY_REQUEST_FORMAT",
            "loc-args" : [ "Jenna", "Frank"]
        }
    }
}
```

当收到包含以上负载的通知时，设备会从合适的应用 `Localizable.strings` 文件中拉取 `GAME_PLAY_REQUEST_FORMAT ` 字符串，然后合入提供的玩家名称。对于一个偏好语言为英语的用户，消息字符串的结果会变成 “Jenna and Frank have invited you to play Monopoly.”。对于其他语言，该字符串会变成消息的其它翻译版本（合并提供的玩家名称）。

当为 `Localizable.strings` 制作消息字符串时，可以使用的格式说明符与用在本地内容的相同。例如，可以使用 `%n$@` 形式的位置说明符来允许重排参数，可以使用 `%%` 说明符来创建一个百分号字母。

有关在通知负载中可以包含的键的额外信息，请看 [Payload Key Reference](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/PayloadKeyReference.html#//apple_ref/doc/uid/TP40008194-CH17-SW1)。更多有关国际化和为应用提供本地化内容的信息，请看 [Internationalization and Localization Guide](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPInternational/Introduction/Introduction.html#//apple_ref/doc/uid/10000171i)。

## 与 APNs 通信

APNs 提供者 API 让你可以发送远程通知请求到 APNs，然后 APNs 把通知传达给 iOS、tvOS、macOS 设备上的应用以及 Apple Watch（通过 iOS）。

提供者 API 基于 HTTP/2 网络协议，每一个交互都以发自提供者的 POST 请求开始，该请求包含一个 JSON 负载和一个设备令牌。APNs 把通知负载转发给指定应用设备上的应用（由请求中包含的设备令牌来识别）。

*提供者*是服务器，由你部署、管理并配置以和 APNs 一起工作的服务器。

### 提供者认证令牌

为了安全地连接到 APNs，可以使用提供者认证令牌或提供者证书。这节讲述使用令牌的连接。

提供者 API 支持 JSON Web Token（JWT）规范，可以连同每条推送通知一起传递表达式和元数据（声明）到 APNs。详细请参考规范 [https://tools.ietf.org/html/rfc7519](https://tools.ietf.org/html/rfc7519)。关于 JWT 的额外信息，连同生成签名 JWTs 的可用库的清单，请看 [https://jwt.io](https://jwt.io/)。

提供者认证令牌是你构造的 JSON 对象，它的头部必须包括：

* 用来加密令牌的加密算法 （alg）
* 一个 10 个字母的*秘钥标识*（kid）键，从 [你的开发者账户](https://developer.apple.com/account/) 获取

令牌的声明负载必须包括：

* 注册声明键 *issuer* (iss)，值为 10 个字母的团队 ID，从 [你的开发者账户](https://developer.apple.com/account/) 获取
* 注册声明键 *issued at* (iat)，值表示令牌生成的时间，按照从 Epoch （1970-01-01 00:00:00）开始到现在经过的秒数， UTC 格式。

创建令牌之后，必须使用私钥签名。然后使用 P-256 曲线的椭圆曲线数字签名算法（ECDSA）和 SHA-256 哈希算法加密令牌。在算法头部键（alg）中指定值  `ES256` 。关于如何配置令牌的信息，请看 Xcode Help [Configure push notifications](http://help.apple.com/xcode/mac/current/#/dev11b059073)。

一个已解密的 JWT 提供者认证令牌有以下格式：
```
{
    "alg": "ES256",
    "kid": "ABC123DEFG"
}
{
    "iss": "DEF123GHIJ",
    "iat": 1437179036
 }
```
> 注意</br>
> APNs 仅支持使用 ES256 算法签名的提供者认证令牌。不安全的或者使用其它算法签名的 JWTs 会被拒绝，提供者会收到 `InvalidProviderToken(403)` 响应。

为了确保安全性，APNs 要求定期地生成新的令牌。新的令牌拥有新的 *issued at* 声明键，它的值表明令牌生成的时间。如果令牌发布的时间戳不在此前一小时以内，APNs 会拒绝后续的推送消息，返回一个 `ExpiredProviderToken(403)` 错误。

如果怀疑提供者的令牌签名秘钥被泄漏，可以从 [你的开发者账户](https://developer.apple.com/account/) 废弃它。可以发行一个新的秘钥对然后使用新的私钥来生成新的令牌。为了最高安全性，关闭所有到 APNs 的连接（使用当前已废弃秘钥签名的令牌的连接），在用新的秘钥签名令牌之后才去重连。

### APNs 提供者证书

提供者证书（按照 Xcode Help 中 [Configure push notifications](http://help.apple.com/xcode/mac/current/#/dev11b059073) 的讲解去获取），能够连接到 APNs 生产和开发环境中。

可使用 APNs 证书发送通知到主应用（由它的 bundle ID 识别），也可以发送到与应用关联的任何 Apple Watch complications 或者后台 VoIP 服务。使用证书中的  `( 1.2.840.113635.100.6.3.6 )` 扩展来为推送通知识别主题。例如，如果你提供一个 bundle ID 为 `com.yourcompany.yourexampleapp` 的应用，你可以在证书中指定以下的主题：
```
Extension ( 1.2.840.113635.100.6.3.6 )
Critical NO
Data com.yourcompany.yourexampleapp
Data app
Data com.yourcompany.yourexampleapp.voip
Data voip
Data com.yourcompany.yourexampleapp.complication
Data complication
```

### APNs 连接

发送远程通知的第一步，建立一条到适当的 APNs 服务器的连接：
* **开发服务器**：api.development.push.apple.com:443
* **生产服务器**：api.push.apple.com:443

>注意</br>
>当与 APNs 通信时，可以选择使用 `2197` 端口。你也许会用到这个端口，例如，为了允许 APNs 流量通过防火墙但阻止其它 HTTPS 流量。

当连接到 APNs 时，提供者必须支持 TLS 1.2 或更高版本。可以使用提供者客户端证书（从 [你的开发者账户](https://developer.apple.com/account/) 获取的），就像 [Creating a Universal Push Notification Client SSL Certificate](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html#//apple_ref/doc/uid/TP40012582-CH26-SW11) 描述的那样。

为了不使用 APNs 提供者证书进行连接，你必须创建提供者认证令牌（通过 [你的开发者账户](https://developer.apple.com/account/) 发布的秘钥签名的令牌，请看 Xcode Help 中的 “[Configure push notifications](http://help.apple.com/xcode/mac/current/#/dev11b059073)”）。在拿到这个令牌之后，就可以开始发送推送消息了。之后你必须定期地更新令牌，每一个 APNs 提供者认证令牌都有为期一小时的有效期。

APNs 允许每条连接有多个并发流。流的具体数目根据使用的提供者证书或认证令牌有所不同，根据不同的服务器负载也会有所不同。不要假定流的具体数目。

当使用令牌而不是证书来建立到 APNs 的连接时，连接上只允许一个流直到使用有效的提供者认证令牌发送推送消息。APNs 忽略 `HTTP/2 PRIORITY` 帧，所以不要在流上发送这些帧。

#### 管理连接的最佳实践

在多个通知中保持到 APNs 的连接为打开状态，不要反复地打开和关闭连接。APNs 把快速的打开和断开连接当成拒绝服务攻击。应该让连接保持打开状态除非你知道它在接下来的一段时间内会空转。例如，如果一天仅给用户发送一次通知，那么每天使用一个新的连接是可以接受的。

不要为每条发送的推送请求生成新的提供者认证令牌。在获得令牌之后，在令牌的有效期内（整整一小时）继续用在所有推送请求上。

可以建立多条到 APNs 服务器的连接来提升性能。当发送大量的远程通知时，通过连接把它们分发到多个服务器终端上。与使用单条连接相比，这可以提升性能（通过让你更快地发送远程通知和让 APNs 更快的分发通知）。

如果提供者证书被废弃了，或者正在使用的用来签名提供者令牌的秘钥被废弃了，那么关闭所有到 APNs 的现有连接然后打开新连接。

可以使用 HTTP/2 `PING` 帧来检测连接的健康状况。

#### 终止 APNs 连接

如果 APNs 决定终止一条 HTTP/2 连接，它会发送 `GOAWAY` 帧。`GOAWAY` 帧的负载的 JSON 数据中包含一个 `reason` 键，该键的值表明连接终止的原因。`reason` 键可能的取值请看 [表 8-6](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW17)。

正常的请求失败不会导致连接的终止。

### APNs 通知 API

APNs 提供者 API 由使用 HTTP/2 `POST` 命令配置和发送的请求和响应组成。使用请求来发送推送通知到 APNs 服务器并使用响应来判定该请求的结果。

#### 到 APNs 的 HTTP/2 请求

使用请求来发送通知到指定用户设备。

**表 8-1** HTTP/2 请求域

|  Name    | Value | 
| :-------: | :---------:| 
|:method|POST|
|:path|/3/device/<device-token>|

<device-token> 参数，为目标设备指定十六进制字节的设备令牌。

APNs 需要使用 HPACK（HTTP/2 头部压缩），它可以避免重复的头部键和值。APNs 为 HPACK 维护一张小动态表。为了避免塞满 APNs HPACK 表和迫使表数据的丢弃，按照以下方式来编码头部（尤其是在发送大量流时）：

* `:path` 应该编码成没有索引的字面量头部域
* `auhorization` 请求头，如果有，应该编码成没有索引的字面量头部域
* 适合 `apns-id`、`apns-expiration`、`apns-collapse-id` 请求头采用的编码会有所区别，取决于它是初次还是后续 POST 请求操作的一部分。如下：
	* 首次发送这些头部时，使用*增量索引*的方式编码以允许头部名称加到动态表中
	* 后续发送这些头部时，编码成*没有索引的字面量头部域*

把其它头部编码成没有索引的字面量头部域。关于头部编码的细节，请看 [tools.ietf.org/html/rfc7541#section-6.2.1](http://tools.ietf.org/html/rfc7541#section-6.2.1) 和 [tools.ietf.org/html/rfc7541#section-6.2.2](http://tools.ietf.org/html/rfc7541#section-6.2.2)。

APNs 会忽略除了表 8-2 中列出的请求头之外的请求头。

**表 8-2** APNs 请求头

| 头部 | 描述 | 
| --- | ----| 
| authorization |授权 APNs 为特定主题发送推送通知的提供者令牌。令牌是 Base64URL-encoded JWT 格式，指定成 `bearer <provider token>`。当使用提供者证书来建立连接，这个请求头被忽略|
| apns-id |用来识别通知的标准的 UUID，如果发送通知出现错误，APNs 使用这个值来向服务器指出该通知。标准形式是 32 个小写字母 16 进制数字，使用连字符分 5 组以 “8-4-4-4-12” 的形式显示。一个 UUID 的例子：`123e4567-e89b-12d3-a456-42665544000`。如果省略这个头部，APNs 会创建一个新的 UUID 并在响应中返回|
| apns-expiration |一个以秒来表示的 UNIX Epoch 日期（UTC）。这个头部表示通知失效并可以被丢弃的日期。如果值不为 0，APNs 会保存通知并至少尝试去传送一次通知，如果首次传送失败它会根据需要重复尝试。如果值为 0，通知会被 APNs 视为马上过期并且不会再保存通知或者尝试传送|
| apns-priority |通知的优先级。指定以下值中的一个：</br> * 10-马上发送推送消息。这种优先级的通知必须出发一个弹框、声音或者目标设备的角标。在包含 `content-available` 键的推送通知中使用这个优先级是错误的用法。</br> * 5-推送消息的发送时间会考虑设备的电量。这种优先级的通知可能会被合并然后一起发射出去，它们会被节流，在某些情况下不会传送。</br>如果省略这个头部，APNs 服务器会把优先级设置为 10。|
| apns-topic |远程通知的主题，通常是应用的 bundle ID。在开发者账户中创建的证书必须为这个主题包含能力（capability）。</br>如果证书包含多个主题，必须给这个头部指定一个值。</br>如果省略这个请求头部并且 APNs 证书中没有指定多个主题，APNs 服务器会把证书的主题当作默认主题</br>如果使用的是提供者令牌而不是证书，必须给这个请求头部指定一个值。你提供的主题应该满足开发者账户中的团队名的需要。|
| apns-collapse-id |拥有相同合并标识符的多个通知会被当做一条单一通知显示给用户。这个键的值必须不能超过 64 字节，更多信息，请看 [Quality of Service, Store-and-Forward, and Coalesced Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW5)。|

消息的主体内容是通知负载的 JSON 字典对象。主体数据必须压缩并且最大尺寸为 4KB（4096 字节）。对于 VoIP 通知，主体数据最大的尺寸是 5KB（5120 字节）。关于主体内容中的键与值的信息，请看 [Payload Key Reference](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/PayloadKeyReference.html#//apple_ref/doc/uid/TP40008194-CH17-SW1)。

#### 来自 APNs 的 HTTP／2 响应

表 8-3 列举了请求的响应格式

**表 8-3** APNs 响应头部

|  头部名    | 值 | 
| ----------- | --------- | 
|apns-id|来自请求的 apns-id 值，如果请求中没有包含该值，服务器会创建一个新的 UUID 并在这个头部中返回|
|:status|HTTP 状态码，状态码的清单请看 [表 8-4](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW15)|

表 8-4 列举了一个请求的可能状态码。这些值会包含在响应的 `:status` 头部中。

**表 8-4** APNs 响应的状态码

|  状态码   | 描述 | 
| -------| ---------| 
|200|成功|
|400|错误请求|
|403|证书或者提供者认证令牌有错误|
|405|请求使用错误的 `:method` 值。仅支持 `POST` 请求|
|410|设备令牌对该主题来说不再有效|
|413|通知负载过大|
|429|服务器收到太多同一设备令牌的请求|
|500|服务器内部错误|
|503|服务器关闭并且不可用|

请求成功，响应的主体为空。请求失败，响应的主体会包含一个 JSON 字典（其中包含表 8-5 中列出的键）。当连接终止时，这个 JSON 数据可能会出现在 `GOAWAY` 帧中。

**表 8-5** APNs JSON 数据键

|  键    | 描述 | 
| ------- | --------| 
|reason|失败的原因。错误码被声明为一个字符串。可能的取值清单，请看[表 8-6](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW17)|
|timestamp|如果 `:status` 头部的值为 410，那么这个键的值为 APNs 上一次确认设备令牌对该主题失效的时间。</br>停止推送通知直到设备向提供者注册一个更迟时间戳的令牌。|

表 8-6 列举了响应的 JSON 负载中 `reason` 键可能包含的错误码。

**表 8-6** APNs JSON 的 `reason` 键的值

|状态码|错误字符串|描述|
| ------- | --------- | ---------| 
|400|BadCollapseId|合并标识符超过最大允许尺寸|
|400|BadDeviceToken|指定的设备令牌错误。请确认请求包含一个有效的令牌并且该令牌与环境匹配|
|400|BadExpirationDate|`apns-expiration` 的值有误|
|400|BadMessageId|`apns-id` 的值有误|
|400|BadPriority|`apns-priority` 的值有误|
|400|BadTopic|`apns-topic`失效|
|400|DeviceTokenNotForTopic|设备令牌与指定的主题不匹配|
|400|DuplicateHeaders|有一个或多个头部重复|
|400|IdleTimeout|空闲超时|
|400|MissingDeviceToken|请求的 `:path` 中没有指定设备令牌。请确认 `:path` 头部包含设备令牌|
|400|MissingTopic|请求的 `apns-topic` 头部未指定，需要指定。在客户端使用支持多个主题的证书进行连接时，apns-topic 头部是必须的|
|400|PayloadEmpty|消息负载为空|
|400|TopicDisallowed|不允许推送通知到本主题|
|403|BadCertificate|证书错误|
|403|BadCertificateEnvironment|客户端证书与环境不匹配|
|403|ExpiredProviderToken|提供者令牌过期了需要生成新的|
|403|Forbidden|指定的动作没有得到允许|
|403|InvalidProviderToken|提供者令牌失效或者令牌签名无法验证|
|403|MissingProviderToken|没有使用证书来连接到 APNs 并且 Authorization 头部缺失（或者没有指定提供者令牌）|
|404|BadPath|请求包含一个错误的 `:path` 值|
|405|MethodNotAllowed|指定的 `:method` 不是 POST|
|410|Unregistered|设备令牌对指定的主题无效。</br>预期的 HTTP/2 状态码为 410，请看[表 8-4](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW15)|
|413|PayloadTooLarge|消息负载过大，请看关于最大负载尺寸的细节 [Creating the Remote Notification Payload](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW1)|
|429|TooManyProviderTokenUpdates|提供者令牌更新太频繁|
|429|TooManyRequests|连续给同一设备令牌发出太多请求|
|500|InternalServerError|出现服务器内部错误|
|503|ServiceUnavailable|服务不可用|
|503|Shutdown|服务器已关闭|

#### APNs 的 HTTP/2 请求及响应的例子

清单 8-1 演示了一个为提供者证书构造的请求的例子

**清单 8-1** 单个主题的证书的请求示例

```
HEADERS
  - END_STREAM
  + END_HEADERS
  :method = POST
  :scheme = https
  :path = /3/device/00fc13adff785122b4ad28809a3420982341241421348097878e577c991de8f0
  host = api.development.push.apple.com
  apns-id = eabeae54-14a8-11e5-b60b-1697f925ec7b
  apns-expiration = 0
  apns-priority = 10
DATA
  + END_STREAM
    { "aps" : { "alert" : "Hello" } }
```

清单 8-2 演示了为提供者认证令牌构建的请求的示例

**清单 8-2**提供者认证令牌的请求示例

```
HEADERS
  - END_STREAM
  + END_HEADERS
  :method = POST
  :scheme = https
  :path = /3/device/00fc13adff785122b4ad28809a3420982341241421348097878e577c991de8f0
  host = api.development.push.apple.com
  authorization = bearer eyAia2lkIjogIjhZTDNHM1JSWDciIH0.eyAiaXNzIjogIkM4Nk5WOUpYM0QiLCAiaWF0I
 jogIjE0NTkxNDM1ODA2NTAiIH0.MEYCIQDzqyahmH1rz1s-LFNkylXEa2lZ_aOCX4daxxTZkVEGzwIhALvkClnx5m5eAT6
 Lxw7LZtEQcH6JENhJTMArwLf3sXwi
  apns-id = eabeae54-14a8-11e5-b60b-1697f925ec7b
  apns-expiration = 0
  apns-priority = 10
  apns-topic = <MyAppTopic>
DATA
  + END_STREAM
    { "aps" : { "alert" : "Hello" } }
 shows a sample request constructed for a certificate that contains multiple topics.
```

清单 8-3 演示了为包含多个主题的证书构建的请求的例子

**清单 8-3**包含多个主题的证书的请求的例子

```
HEADERS
  - END_STREAM
  + END_HEADERS
  :method = POST
  :scheme = https
  :path = /3/device/00fc13adff785122b4ad28809a3420982341241421348097878e577c991de8f0
  host = api.development.push.apple.com
  apns-id = eabeae54-14a8-11e5-b60b-1697f925ec7b
  apns-expiration = 0
  apns-priority = 10
  apns-topic = <MyAppTopic> 
DATA
  + END_STREAM
    { "aps" : { "alert" : "Hello" } }
```

清单 8-4 给出了成功的请求的响应的例子
**清单 8-4** 成功请求的响应例子

```
HEADERS
  + END_STREAM
  + END_HEADERS
  apns-id = eabeae54-14a8-11e5-b60b-1697f925ec7b
  :status = 200
```

清单 8-5 演示了错误发生时的响应的例子

**清单 8-5** 错误发生时的响应的例子

```
HEADERS
  - END_STREAM
  + END_HEADERS
  :status = 400
  content-type = application/json
    apns-id: <a_UUID>
DATA
  + END_STREAM
  { "reason" : "BadDeviceToken" }
```

## 负载的键参考目录

每条通知提供一个负载（负载中有应用特有信息和有关如何传递通知给用户的细节）。负载是在服务器上创建的 JSON 字典对象（定义 [RFC 4627](http://www.ietf.org/rfc/rfc4627.txt)）。JSON 字典对象必须包含 `aps` 键，它的值是一个字典（包含系统用来传递通知的数据）。`aps` 字典的主要内容决定系统是否做以下的事情：

* 显示提示消息给用户
* 给应用图标显示角标
* 播放一段声音
* 静默地传递通知

除了 `aps` 字典，JSON 字典可以包含应用特有内容的自定义键值。自定义值必须使用 JSON 结构并且只能使用基本数据类型，例如：字典（对象）、数组、字符串、数字和布尔类型。不要在负载中包含客户的信息或任何敏感数据，除非数据经过加密或者数据在应用之外是无用的。例如，一个即时通讯应用可以包含会话标识，之后可以用来找出相应的用户会话。通知的数据永远不该有破坏性，也就是说，应用永远不该使用通知来删除用户设备上的数据。

当使用基于 HTTP/2 的 APNs 提供者 API 时，最大的 JSON 字典尺寸是 4KB。对于遗留的 API，负载的最大尺寸较小。

关于如何创建负载的信息及例子，请看 [Creating the Remote Notification Payload](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW1)。

### APS 字典键

`aps` 字典包含苹果用来传递通知到用户设备的键。键指定了在提醒用户时想让系统使用的交互类型。表 9-1 列举了字典中包含的键与每一个键的类型。aps 字典中其它的键会被苹果忽略。

**表 9-1** `aps`字典的键和值

|键|值类型|注解|
| ------- | ---------| --------- | 
|alert|Dictionary or String|当你想让系统显示标准的弹框或横幅时，包含这个键。应用的通知设置决定了弹框或横幅是否显示。</br>这个键的推荐值为一个字典，字典的键在[表 9-2](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/PayloadKeyReference.html#//apple_ref/doc/uid/TP40008194-CH17-SW5)列出。如果给这个键指定一个字符串的值，该字符串会被当做弹框或横幅的消息文本显示。</br>不支持 JSON `\u` 符号，改为在提示文本中放入实际的 UTF-8 字母|
|badge|Number|当想让系统修改应用图标的角标时包含这个键。</br>如果字典中不含这个键，角标不变。要移除角标，把这个键的值设置为 0。|
|sound|String|当想让系统播放音频时包含这个键。这个键的值为应用 main bundle 中或应用数据容器的 `Library/Sounds` 文件夹中的音频文件名。如果无法找到音频文件或者该值指定为 `default`，系统会播放默认的提示音。关于为通知提供音频文件的细节，请看[Preparing Custom Alert Sounds](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SupportingNotificationsinYourApp.html#//apple_ref/doc/uid/TP40008194-CH4-SW10)。|
|content-available|Number|包含这个键（值为 1）以配置后台更新通知。当这个键出现，系统在后台唤醒你的应用并传递通知到应用的代理。关于配置和处理后台更新通知的信息，请看[Configuring a Background Update Notification](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW8)。|
|category|String|为这个键提供一个字符串值代表通知的类型。这个值与应用已注册分类中的某个分类的 `identifier` 属性的值相对应。了解更多使用自定义动作的信息，请看[Configuring Categories and Actionable Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SupportingNotificationsinYourApp.html#//apple_ref/doc/uid/TP40008194-CH4-SW26)。|
|thread-id|String|为这个键提供一个字符串值代表用来给通知分组的应用特有的标识符。如果提供通知内容应用扩展，可以使用这个值来把通知合并到一起。对于本地通知来说，这个键与 [UNNotificationContent](https://developer.apple.com/documentation/usernotifications/unnotificationcontent) 对象的 [threadIdentifier](https://developer.apple.com/documentation/usernotifications/unnotificationcontent/1649860-threadidentifier) 属性相对应。|

### 弹框提示的键

表 9-2 列举了 `alert` 字典的键和预期的值

**表 9-2** `alert` 属性的子属性

|键|值类型|注解|
| ------- | ---------| ---------| 
|title|String|用来描述通知目的的短字符串。Apple Watch 把这个字符串当作通知界面的一部分来显示。这个字符串仅仅短暂地显示，因而应当精心制作以便可以被快速理解。这个键是在 iOS 8.2 添加的。|
|body|String|提示消息的文本|
|title-loc-key|String or null|当前本地化 `Localizable.strings` 文件的标题字符串的键。这个键的字符串可以使用 `%@` 和 `%n$@` 说明符来格式化以获取在 `title-loc-args` 数组中指定的变量。更多信息请看[Localizing the Content of Your Remote Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW9)。这个键是在 iOS 8.2 添加的。|
|title-loc-args|Array of strings or null|变量字符串的值用来替换 `title-loc-key` 中的格式说明符。更多信息请看[Localizing the Content of Your Remote Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW9)。这个键是在 iOS 8.2 添加的。|
|action-loc-key|String or null|如果为这个键指定字符串，系统会显示包含 “Close” 和 “View” 按钮的弹框。这个字符串会被当成键来从当前本地化中获取本地化字符串，然后用来当做右按钮的标题而不是“View”。更多信息请看 [Localizing the Content of Your Remote Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW9)。|
|loc-key|String|当前本地化 `Localizable.strings` 文件的提示消息字符串的键（由用户语言偏好设置）。这个键的字符串可以使用 `%@` 和 `%n$@` 说明符来格式化以获取在 `loc-args` 数组中指定的变量。更多信息请看 [Localizing the Content of Your Remote Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW9)。|
|loc-args|Array of strings|变量字符串的值用来替换 `loc-key` 中的格式说明符。更多信息请看 [Localizing the Content of Your Remote Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW9)。|
|launch-image|String|应用包中的图片文件名（包含或不含文件扩展名）。当用户点击动作按钮或者移动动作滑动条时这张图片用来当做启动图。如果这个属性没有指定，系统要么使用之前的截图要么使用 `Info.plist` 文件中的 `UILaunchImageFile` 键所指定图片，或者用回 `Default.png`。|

# 遗留信息

## 二进制提供者 API

遗留的二进制接口需要提供者服务器采用本附录所描述的二进制 API。所有的开发者都应当把远程通知提供者服务器迁移到 [Communicating with APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW1) 中所描述的更给力的更高效基于 HTTP/2 的 API。

官方不推荐使用，此节不翻译，有需要请移步[官方文档](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/BinaryProviderAPI.html#//apple_ref/doc/uid/TP40008194-CH13-SW1)。

## 遗留的通知格式

新的开发应当使用现代格式来连接到 APNs，正如 [Communicating with APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW1) 中所描述。

官方不推荐使用，此节不翻译，有需要请移步[官方文档](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/BinaryProviderAPI.html#//apple_ref/doc/uid/TP40008194-CH13-SW1)。
