---
title: iOS 的 main.m 文件
date: 2014-07-13 21:15:12
tags: iOS
---

C语言的入口函数是 main()，Objective-C 也一样。

在项目导航面板中选中 main.m ，可以看到

```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

UIApplicationMain 函数会创建一个 UIApplication 对象。每个 iOS 应用都有且只有一个 UIApplication 对象（单例），该对象的作用时维护运行循环。一旦程序创建了某个 UIApplication 对象，该对象的运行循环就会一直循环下去，main() 的执行也会因此阻塞。

此外，UIApplicationMain 函数还会创建某个指定类(此处为 AppDelegate)的对象，并将其设置为 UIApplication 对象的 delegate。UIApplicationMain 函数的第三个实参为 NSString 类型，指定了该对象所属的类。UIApplication 的 delegate 都需要遵守 UIApplicationDelegate 协议。

`@interface AppDelegate : UIResponder <UIApplicationDelegate>`

在应用启动运行循环时，UIApplication 对象会在应用出现相应状态变化时，向其 delegate 发送特定的消息。如：

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;
- (void)applicationWillResignActive:(UIApplication *)application;
- (void)applicationDidEnterBackground:(UIApplication *)application; 
- (void)applicationWillEnterForeground:(UIApplication *)application;
- (void)applicationDidBecomeActive:(UIApplication *)application;
- (void)applicationWillTerminate:(UIApplication *)application;
```

UIApplication 负责建立应用程序的事件循环（Event Loop），事件循环中可以不断接收交互操作，比如屏幕触摸手势、各类传感器（重力加速器、陀螺仪等）等。