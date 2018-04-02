---
title: Core Bluetooth Programming Guide 译文
date: 2018-03-26 19:09:15
tags: Translate
---
# 介绍

## 关于 Core Bluetooth
Core Bluetooth 框架提供 iOS 应用和 Mac 应用与设备（配备了蓝牙低能耗无线技术的设备）通信的类。例如，应用可以发现、探测并与低能耗外围设备（比如心率监听器和数字恒温器）交互。从 macOS 10.9 和 iOS 6 开始，Mac 和 iOS 设备还可以当做蓝牙低能耗外设来使用，为其它设备提供数据，包括其他 Mac 和 iOS 设备。

<img id='core_bluetooth_architecture' style='width:150px;height:200px' src="https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBTechnologyFramework_2x.png" />

## 一览

蓝牙低能耗无线技术基于蓝牙 4.0 规范，规范中除了别的之外，定义了与低能耗设备通信的一套协议。Core Bluetooth 框架是蓝牙低能耗协议栈的一个抽象，也就是说，它为开发者隐藏了许多规范中的底层细节，让开发者更加容易开发应用（与蓝牙低能耗设备交互的应用）。

### 中央和外围是 Core Bluetooth 的核心成员

在蓝牙低能耗通信中，有两个核心成员：中央（central）和外围（peripheral）。每个成员扮演不同的角色。外围通常拥有其他设备需要的数据，中央通常使用外围提供的信息来完成一些任务。例如，一个配备了蓝牙低能耗技术的数字恒温器可能为一个 iOS 应用提供房间的温度信息，然后该应用采用用户友好的方式来显示温度。

每个成员在扮演它的角色时都会执行一组不同的任务。外围通过在空中广播持有的数据来让自身的存在被感知，中央设备扫描附近的外围设备（可能包含中央设备感兴趣的数据）。当中央设备发现外围设备，中央设备就请求与外围设备连接并开始探测和交互外围设备的数据。外围设备负责以适当的方式来响应中央设备。

> 相关章节：[Core Bluetooth Overview](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothOverview/CoreBluetoothOverview.html#//apple_ref/doc/uid/TP40013257-CH2-SW1)

### Core Bluetooth 简化了一般的蓝牙任务

Core Bluetooth 框架抽离了蓝牙 4.0 规范中的底层细节。因此，应用中需要实现的一般蓝牙低能耗任务被简化了。如果开发实现中央角色的应用，Core Bluetooth 使得发现、连接外围设备和探测、交互外围数据变得简单。另外，Core Bluetooth 还让本地设备实现外围角色变得简单。

> 相关章节：[Performing Commmon Central Role Tasks](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW1),[Performing Common Peripheral Role Tasks](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonPeripheralRoleTasks/PerformingCommonPeripheralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH4-SW1)

### iOS 应用的状态影响蓝牙的表现

当应用处于后台或挂起状态时，蓝牙相关的特性会受到影响。在这两种状态下，默认是应用无法执行蓝牙低能耗任务。也就是说，如果应用需要在后台执行蓝牙低能耗任务，可以声明支持 Core Bluetooth 后台运行模式中的一个或两个（一个属于中央角色，另一个属于外围角色）。即使在你指定了一个后台运行模式或两个都指定，当应用处于后台时，某些蓝牙任务的执行依然会有所不同，设计应用时，需要考虑到这些差异。

即使应用支持后台处理，应用仍然可能在任意时刻被系统终止以清空内存给当前前台应用使用。在 iOS 7之后，Core Bluetooth 支持保存中央和外围管理者对象的状态信息并在应用启动的时候恢复该状态，可以使用这个特性来支持涉及蓝牙设备的长期活动（long-term actions）。

> 相关章节：[Core Bluetooth Background Processing for iOS Apps](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html#//apple_ref/doc/uid/TP40013257-CH7-SW1)

### 遵循最佳实践以增强用户体验

Core Bluetooth 框架使应用可以控制许多常见的蓝牙低能耗事务。遵循最佳实践，以负责任的方式利用这种级别的控制，增强用户体验。

例如，当实现中央和外围角色时所执行的许多任务会使用设备的机载无线电来在空中传播信号。因为设备的无线电与其它形式的无线通信是共享的，并且无线电的使用会给设备的电池寿命带来不利影响，所以，设计应用总当减少使用无线电。

> 相关章节：[Best Practices for Interacting with a Remote Peripheral Device](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW1),[Best Practices for Setting Up Your Local Device as a Peripheral](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral.html#//apple_ref/doc/uid/TP40013257-CH5-SW1)

## 如何使用这个文档

如果从未用过 Core Bluetooth 框架，或者不熟悉基本的蓝牙低能耗概念，请通读这个文档。在 [Core Bluetooth 概述](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothOverview/CoreBluetoothOverview.html#//apple_ref/doc/uid/TP40013257-CH2-SW1)中，你可以学到关键的术语和概念。

在理解了关键的概念之后，请阅读 [执行常见中央角色任务](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW1) 来学习如何开发应用以在本地设备实现中央角色。类似的，学习如何开发应用以在本地设备中实现外围角色，请看 [执行常见外围角色任务](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonPeripheralRoleTasks/PerformingCommonPeripheralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH4-SW1)。

为确保应用运行良好并遵循最佳实践，请阅读后面章节：[Core Bluetooth Background Processing for iOS Apps](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html#//apple_ref/doc/uid/TP40013257-CH7-SW1)、[Best Practices for Interacting with a Remote Peripheral Device](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW1) 和 [Best Practices for Setting Up Your Local Device as a Peripheral](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral.html#//apple_ref/doc/uid/TP40013257-CH5-SW1)

## 另请参阅

官方 [Bluetooth Special Interest Group (SIG) website](http://www.bluetooth.org/) 提供有关蓝牙低能耗无线技术的权威信息，在那可以找到 [蓝牙 4.0 规范](https://www.bluetooth.org/en-us/specification/adopted-specifications)。

如果正在设计使用蓝牙低能耗技术与苹果产品通信的硬件配件（包括 Mac、iPhone、iPad 和 iPod touch 模型），请看 [Bluetooth Accessory Design Guidelines for Apple Products](https://developer.apple.com/hardwaredrivers/BluetoothDesignGuidelines.pdf)。如果你的蓝牙配件（使用蓝牙低能耗与 iOS 设备连接的配件）需要存取 iOS 设备产生的通知，请看 [Apple Notification Center Service (ANCS) Specification](https://developer.apple.com/library/content/documentation/CoreBluetooth/Reference/AppleNotificationCenterServiceSpecification/Introduction/Introduction.html#//apple_ref/doc/uid/TP40013460)。

# Core Bluetooth 概述

Core Bluetooth 框架使得 iOS 和 Mac 的应用可以和蓝牙低能耗设备通信。例如，你的应用可以发现、探测并与低能耗外围设备进行交互，比如心率监听器、数字恒温器甚至是其它 iOS 设备。

该框架是蓝牙 4.0 规范的一个抽象，用于低能耗设备。也就是说，它对开发者隐藏了许多规范中的底层细节，使得开发与蓝牙低能耗设备交互的应用变得简单。因为该框架基于规范，所以一些来自规范的概念和术语被本文采纳。本章介绍了使用 Core Bluetooth 框架开发优秀应用需要知道的关键术语和概念。

> 重要：iOS 10.0 及之后的应用必须在 `Info.plist` 文件中包含需要存取的数据类型的使用描述键，否则会崩溃。存取蓝牙外围数据，必须包含 [NSBluetoothPeripheralUsageDescription](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW20)。

## 中央和外围设备以及它们在蓝牙通信中的角色

在所有的蓝牙低能耗通信中，存在两个主要角色：中央设备和外围设备。基于有点类似传统的 `客户端-服务器` 结构，*外围设备* 通常拥有其他设备需要的数据。*中央设备* 通常使用外围设备提供的信息来完成一些特别的任务。如图 1-1 所示，一个心率监听器可能拥有 Mac 或 iOS 应用所需的有用信息（为了以用户友好的方式来显示用户心率）。

**图 1-1** 中央和外围设备

![中央和外围设备](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBDevices1_2x.png)

### 中央设备发现和连接正在广播的外围设备

外围设备以广播数据包的形式来广播数据。*广播数据包* 是一包相对小的数据，可能包含外围设备必须提供的有用信息，比如外围设备的名称和主要功能。例如，一个数字恒温器可能广播一个房间的当前温度。在蓝牙低能耗中，广播是外围设备让自身存在被感知的主要方式。

另一方面，中央设备可以扫描并监听任何正在广播它所感兴趣的信息的外围设备。如图 1-2 所示，中央设备可以请求连接到它所发现的任何外围设备。

**图 1-2** 广播和发现

![广播和发现](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/AdvertisingAndDiscovery_2x.png)

### 外围设备的数据是如何组织的

连接外围设备的目的是探测和交互数据。然而，在这之前，需要理解外围设备的数据是如何组织的。

外围设备可能包含一个或者多个服务，或者提供有关它们连接信号强度的有用信息。*服务（service）* 是一批数据以及关联的行为用来实现设备的功能和特征（或者设备的一部分）。例如，心率监听器的一个服务可能是公开来自监听器心率传感器的心率数据。

服务本身由特性或者内部服务（也就是，引用其它服务）组成。*特性（characteristic）* 提供有关外围服务的更深层次的细节。例如，刚才所说的心率服务可能包含一个描述心率传感器的预期身体位置的特性以及其它传递心率测量数据的特性。图 1-3 演示了心率监听器的服务和特性的一个可能结构。

图 1-3 外围设备的服务和特性

<img id='peripheral_service_characteristic' style='width:200px;height:350px' src="https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBPeripheralData_Example_2x.png" />

### 中央设备探索并与外围设备数据进行交互

在中央设备与外围设备成功建立连接之后，它可以发现外围设备提供的全方位服务和特性（广播数据可能只包含可用服务的一部分）。

通过读或者写服务的特性的值，中央设备可以与外围设备服务交互。例如，应用可能从数字恒温器请求当前的房间温度，或者给恒温器提供一个值用来设置房间温度。

## 中央设备和蓝牙设备的数据是如何表示的

在蓝牙低能耗通信中所涉及到的角色和数据都被以一种简单、直接的方式映射到 Core Bluetooth 框架中。

### 中央侧的对象

当使用本地中央设备与远程外围设备进行交互时，是在执行蓝牙低能耗通信的中央侧上的活动。除了建立本地外围设备（并用它来响应中央设备的请求）之外，大部分的蓝牙事务都发生在中央侧。

有关如何在应用中实现中央角色的信息，请看 [Performing Common Central Role Tasks](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW1) 和 [Best Practices for Interacting with a Remote Peripheral Device](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW1)。

#### 本地中央设备和远程外围设备

在中央设备侧，本地中央设备由 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 对象表示。这些对象用来管理已发现或者已连接的远程外围设备（由 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 对象表示），包括扫描、发现以及连接到广播外围设备。图 1-4 演示了在 Core Bluetooth 框架中本地中央设备和远程外围设备是如何表示的。

**图 1-4** 中央设备侧 Core Bluetooth 对象
![中央设备侧 Core Bluetooth 对象](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBObjects_CentralSide_2x.png)

#### 远程外围设备数据由 CBService 和 CBCharacteristic 对象表示

当与远程外围设备进行数据交互时，是在处理它的服务和特性。在 Core Bluetooth 框架中，远程外围设备的服务由 [CBService](https://developer.apple.com/documentation/corebluetooth/cbservice) 对象表示。类似地，远程外围设备的服务的特性由 [CBCharacteristic](https://developer.apple.com/documentation/corebluetooth/cbcharacteristic) 对象表示。图 1-5 演示了远程外围设备的服务和特性的基本结构。

**图 1-5** 远程外围设备的服务特性树
![远程外围设备的服务特性树](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/TreeOfServicesAndCharacteristics_Remote_2x.png)

### 外围侧的对象

在 macOS 10.9 和 iOS 6 之后，Mac 和 iOS 设备可以当做蓝牙低能耗外围设备来使用，为其它设备提供数据，包括其他 Mac、iPhone 和 iPad 设备。当设置你的设备以实现外围角色时，是在执行蓝牙低能耗通信外围设备侧的活动。

#### 本地外围设备和远程中央设备

在外围设备侧，本地外围设备由 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 对象表示。这些对象用来管理本地外围设备服务和特性数据库内的已发布服务，并向远程中央设备（由 [CBCentral](https://developer.apple.com/documentation/corebluetooth/cbcentral) 对象表示）广播这些服务。外围设备管理者对象也用来响应来自这些远程中央设备的读写请求。图 1-6 演示了在 Core Bluetooth 框架中如何表示本地外围设备和远程中央设备。

图 1-6 外围设备侧 Core Bluetooth 对象

![外围设备侧 Core Bluetooth 对象](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBObjects_PeripheralSide_2x.png)

#### 本地外围设备数据由 CBMutableService 和 CBMutableCharacteristic 对象表示

当建立并与本地外围设备（由 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 对象表示）交互数据时，是在处理它的服务和特性的可变版本。在 Core Bluetooth 框架中，本地外围设备的服务由 [CBMutableService](https://developer.apple.com/documentation/corebluetooth/cbmutableservice) 对象表示。类似地，本地外围设备的服务的特性由 [CBMutableCharacteristic](https://developer.apple.com/documentation/corebluetooth/cbmutablecharacteristic) 对象表示。图 1-7 演示了本地外围设备的服务和特性的基本结构。

**图 1-7** 本地外围设备的服务特性树

<img style="width:300px;height:250px" src='https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/TreeOfServicesAndCharacteristics_Local_2x.png'>

有关如何设置本地设备以实现外围角色的信息，请看 [Performing Common Peripheral Role Tasks](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonPeripheralRoleTasks/PerformingCommonPeripheralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH4-SW1) 和 [Best Practices for Setting Up Your Local Device as a Peripheral](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral.html#//apple_ref/doc/uid/TP40013257-CH5-SW1)。

# 执行常见的中央角色任务

实现了蓝牙低能耗通信中央角色的设备执行一些常见的任务，例如，发现和连接可用的外围设备，探测并与外围设备数据进行交互。实现了外围角色的设备也执行一些常见但不同的任务，例如，发布和广播服务并响应来自已连接中央设备的读、写以及订阅请求。

在本章，你将学到如何使用 Core Bluetooth 框架来执行最常见的来自中央设备侧的蓝牙低能耗任务类型。下面的代码例子会帮助你开发应用以在本地设备上实现中央角色。特别地，你将学到如何：

* 开启一个中央管理者对象
* 发现并连接到正在广播的外围设备
* 连接外围设备之后，探索它的数据
* 给外围设备的服务的特性值发送读写请求
* 订阅特性值以在值被更新时收到通知

在下一章节，你将学到如何在本地设备开发应用以实现外围角色。

本章的代码例子简单并且抽象，需要适当的修改以合并到真实应用中使用。实现中央角色的更高级的专题（包括小窍门、技巧以及最佳实践），囊括在后面的章节中，[Core Bluetooth Background Processing for iOS Apps](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html#//apple_ref/doc/uid/TP40013257-CH7-SW1) 和 [Best Practices for Interacting with a Remote Peripheral Device](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW1) 。


## 开启一个中央管理者

因为 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 对象是本地中央设备的 Core Bluetooth 面向对象的表示，在执行任何蓝牙低能耗事务之前需要分配并初始化中央管理者实例，通过调用它的 [initWithDelegate:queue:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1519001-init) 方法来初始化。

```
myCentralManager = [[CBCentralManager alloc] initWithDelegate:self queue:nil options:nil];
```

在这个例子中，`self` 被设置为代理以接受中央角色事件。通过指定派发队列为 `nil` ，中央管理者会使用主队列来派发中央角色事件。

当创建中央管理者时，中央管理者会调用它的代理对象的 [centralManagerDidUpdateState:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518888-centralmanagerdidupdatestate) 方法。必须实现这个代理方法以确保中央设备支持蓝牙低能耗并且可用。有关如何实现这个代理方法的信息，请看 [*CBCentralManagerDelegate Protocol Reference*](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate)。

## 发现正在广播的外围设备

一旦初始化，中央管理者的第一任务就是发现外围设备。如 [Central Discover and Connect to Peripherals That Are Advertising](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothOverview/CoreBluetoothOverview.html#//apple_ref/doc/uid/TP40013257-CH2-SW3) 中提到的，外围设备通过广播使得自身的存在被感知。应用通过调用中央管理者的 [scanForPeripheralsWithServices:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518986-scanforperipherals) 方法来发现附近正在广播的外围设备。

```
[myCentralManager scanForPeripheralsWithServices:nil options:nil];
```

> **注意** ：如果第一个参数指定为 `nil`，中央管理者会返回 *所有* 发现的外围设备，无视它们支持的服务。在真实的应用中，通常会指定一个包含 [CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) 对象的数组，每一个都表示外围设备正在广播的服务的全局唯一标识符（UUID）。当指定一个包含服务 UUID 的数组时，中央管理者仅返回广播那些服务的外围设备，以便让你只扫描让你感兴趣的设备。</br></br>
> UUID 以及表示它们的 [CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) 对象，在 [Services and Characteristics Are Identified by UUIDs](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonPeripheralRoleTasks/PerformingCommonPeripheralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH4-SW8) 中有详细的讨论。

每当中央管理者发现了外围设备，它会调用它的代理对象的 [centralManager:didDiscoverPeripheral:advertisementData:RSSI:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518937-centralmanager) 方法，最新发现的外围设备会被当成一个 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 对象返回。如果打算连接到已发现的外围设备，那么保留一个对它的强引用以防系统把它回收。下面的例子演示了使用类属性来维持引用已发现的外围设备的场景：

```
- (void)centralManager:(CBCentralManager *)central
 didDiscoverPeripheral:(CBPeripheral *)peripheral
     advertisementData:(NSDictionary *)advertisementData
                  RSSI:(NSNumber *)RSSI {
 
    NSLog(@"Discovered %@", peripheral.name);
    self.discoveredPeripheral = peripheral;
    ...
```

如果期望连接到多台设备，则使用 `NSArray` 来保留已发现外围设备。无论如何，一旦找到了感兴趣的所有外围设备，停止扫描其它设备以节省电量。

```
[myCentralManager stopScan];
```

## 在发现外围设备之后连接它

在发现了正在广播让你感兴趣的服务的外围设备之后，通过调用中央管理者的 [connectPeripheral:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518766-connect) 方法来发起一个连接外围设备的请求，指明想要连接的外围设备：

```
[myCentralManager connectPeripheral:peripheral options:nil];
```

如果连接请求成功，中央管理者会调用它的代理对象的 [centralManager:didConnectPeripheral:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518969-centralmanager) 方法。在开始与外围设备交互之前，设置它的代理以确保代理收到恰当的回调：

```
- (void)centralManager:(CBCentralManager *)central
  didConnectPeripheral:(CBPeripheral *)peripheral {
 
    NSLog(@"Peripheral connected");
    peripheral.delegate = self;
    ...
```

## 发现所连接的外围设备的服务

在建立了到外围设备的连接之后，可以探索它的数据。探索外围设备提供的数据的第一步是发现它的可用服务。因为外围设备广播的数据数量有大小限制，你也许会发现外围设备的服务比它所广播（在它的广播数据包中）的要多得多。通过调用外围设备的 [discoverServices](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518706-discoverservices) 方法来发现它所提供的服务，像这样：

```
[peripheral discoverServices:nil];
```

> **注意**：在一个真实的应用中，通常不会传入 `nil` 作为参数，因为这么做会返回外围设备上 *所有* 可用的服务。因为外围设备可能包含很多服务（多于你所感兴趣的服务），发现所有的服务会浪费电量并且浪费时间。相反，你通常指定你所感兴趣的服务的 UUID，如 [Explore a Peripheral's Data Wisely](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW6）中所示。

当指定的服务被找到，外围设备（你所连接的 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 对象）会调用它的代理对象的 [peripheral:didDiscoverServices:](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate/1518744-peripheral) 方法。Core Bluetooth 创建一个包含 [CBService](https://developer.apple.com/documentation/corebluetooth/cbservice) 对象的数组，每一个与外围设备上发现的服务对应。如下所示，你可以实现这个代理方法以访问已发现服务数组：

```
- (void)peripheral:(CBPeripheral *)peripheral
didDiscoverServices:(NSError *)error {
 
    for (CBService *service in peripheral.services) {
        NSLog(@"Discovered service %@", service);
        ...
    }
    ...
```

## 发现服务的特性

当找到感兴趣的服务，下一步就是探索外围设备提供的所有服务特性。发现服务所有特性的方法很简单，就是调用外围设备的 [discoverCharacteristics:forService:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518797-discovercharacteristics) 方法，指定合适的服务，像这样：

```
NSLog(@"Discovering characteristics for service %@", interestingService);
    [peripheral discoverCharacteristics:nil forService:interestingService];
```

> **注意**：在一个真实的应用中，通常不会传入 `nil` 作为参数，因为这么做会返回外围设备服务的所有特性。因为外围设备可能包含更多比你感兴趣的特性，发现所有的特性浪费电量并且浪费时间。相反，通常指定感兴趣的特性的 UUID。

当指定的服务的特性被发现时，外围设备会调用它的代理对象的 [peripheral:didDiscoverCharacteristicsForService:error:](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate/1518821-peripheral) 方法。Core Bluetooth 会创建一个包含 [CBCharacteristic](https://developer.apple.com/documentation/corebluetooth/cbcharacteristic) 对象的数组，每一个都与已发现的特性对应。下面的例子演示了如何实现这个代理方法：

```
- (void)peripheral:(CBPeripheral *)peripheral
didDiscoverCharacteristicsForService:(CBService *)service
             error:(NSError *)error {
 
    for (CBCharacteristic *characteristic in service.characteristics) {
        NSLog(@"Discovered characteristic %@", characteristic);
        ...
    }
    ...
```

## 取回特性的值

特性包含一个单一值，用来表示有关外围设备服务的信息。例如，一个健康体温计的温度测量特性可能有一个值表示温度（以摄氏度来表示）。可以通过直接读取或订阅的方式来取回特性的值。

### 读取特性的值

在找到感兴趣的服务的特性之后，通过调用外围设备的 [readValueForCharacteristic:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518759-readvalue) 方法可以读取特性的值，指定合适的特性，像这样：

```
NSLog(@"Reading value for characteristic %@", interestingCharacteristic);
    [peripheral readValueForCharacteristic:interestingCharacteristic];
```

当试图读取特性的值的时候，外围设备会调用它的代理对象的 [peripheral:didUpdateValueForCharacteristic:error:](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate/1518708-peripheral) 方法取回数值，如果成功取回，就可以通过特性的 `value` 属性来访问，像这样：

```
- (void)peripheral:(CBPeripheral *)peripheral
didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic
             error:(NSError *)error {
 
    NSData *data = characteristic.value;
    // parse the data as needed
    ...
```

> **注意**：不是所有的特性都是可读的。可以通过检查它的 [properties](https://developer.apple.com/documentation/corebluetooth/cbcharacteristic/1519010-properties) 属性是否包含 [CBCharacteristicPropertyRead](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicproperties/1519104-read) 常量来判断特性是否可读。如果尝试读取一个不可读的特性，[peripheral:didUpdateValueForCharacteristic:error:](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate/1518708-peripheral) 代理方法会返回相配的错误。

### 订阅特性值
对静态值来说，通过使用 [readValueForCharacteristic:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518759-readvalue) 方法来读取特性的值可能很高效，但对于动态值来说这种方式可能不是最高效的。通过订阅来获取随时间而改变的特性值，例如，心率。当订阅了特性值，在值改变之后会收到来自外围设备的通知。

通过调用 [setNotifyValue:forCharacteristic:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518949-setnotifyvalue) 方法来订阅感兴趣的特性值，指定第一个参数为 `YES`，像这样：

```
[peripheral setNotifyValue:YES forCharacteristic:interestingCharacteristic];
```

当订阅或取消订阅特性值，外围对象会调用它的代理对象的 [peripheral:didUpdateNotificationStateForCharacteristic:error:](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate/1518768-peripheral) 方法。如果有任何原因导致订阅请求失败，你可以实现这个代理方法来访问引发错误的原因，如下所示：

```
- (void)peripheral:(CBPeripheral *)peripheral
didUpdateNotificationStateForCharacteristic:(CBCharacteristic *)characteristic
             error:(NSError *)error {
 
    if (error) {
        NSLog(@"Error changing notification state: %@",
           [error localizedDescription]);
    }
    ...
```

>**注意**：并非所有的特性都提供订阅。可以通过检查它的 [properties](https://developer.apple.com/documentation/corebluetooth/cbcharacteristic/1519010-properties) 属性是否包含 [CBCharacteristicPropertyNotify](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicproperties/1518976-notify) 或者 [CBCharacteristicPropertyIndicate](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicproperties/1519085-indicate) 常量来判断特性是否支持订阅。

在成功订阅特性值之后，外围设备会在值发生改变时通知应用。每一次值变化，外围对象会调用它的代理对象的 [peripheral:didUpdateValueForCharacteristic:error:](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate/1518708-peripheral) 方法。为了获取已更新的值，可以按照上面 [Reading the Value of a Characteristic](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW12) 所说的同样的方式来实现这个方法。

## 写特性值

有时写特性值很有意义。例如，如果应用与蓝牙低能耗数字恒温器进行交互，你可能想要给恒温器提供一个值，用来设置房间的温度。如果一个特性值是可写的，可以通过调用外围设备的 [writeValue:forCharacteristic:type:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518747-writevalue) 方法使用一个 [NSData](https://developer.apple.com/documentation/foundation/nsdata) 对象来写入值，像这样：

```
NSLog(@"Writing value for characteristic %@", interestingCharacteristic);
    [peripheral writeValue:dataToWrite forCharacteristic:interestingCharacteristic
        type:CBCharacteristicWriteWithResponse];
```

当写特性值时，指定要执行的写类型。在上面的例子中，写类型为 [CBCharacteristicWriteWithResponse](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicwritetype/1519131-withresponse)，指示外围设备通过调用它的代理对象 [peripheral:didWriteValueForCharacteristic:error:](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate/1518823-peripheral) 方法来让应用知道是否写成功。实现这个代理方法来处理错误条件，如下所示：

```
- (void)peripheral:(CBPeripheral *)peripheral
didWriteValueForCharacteristic:(CBCharacteristic *)characteristic
             error:(NSError *)error {
 
    if (error) {
        NSLog(@"Error writing characteristic value: %@",
            [error localizedDescription]);
    }
    ...
```

相反，如果指定写类型为 [CBCharacteristicWriteWithoutResponse](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicwritetype/1518922-withoutresponse)，写请求会以 `尽力而为` 的方式来执行，并且传递没有保证也没有反馈，外围设备不会调用任何代理方法。有关 Core Bluetooth 框架中支持的写类型信息，请看 [CBPeripheral Class Reference](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 中的枚举 [CBCharacteristicWriteType](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicwritetype)。

> **注意**：特性可能仅支持某些写类型或者根本不支持。通过检查 [properties](https://developer.apple.com/documentation/corebluetooth/cbcharacteristic/1519010-properties) 属性是否包含 [CBCharacteristicPropertyWriteWithoutResponse](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicproperties/1518734-writewithoutresponse) 或 [CBCharacteristicPropertyWrite](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicproperties/1519089-write) 常量来判断特性所支持的写类型。

# 执行常见的外围角色任务

在上一章节中，学习了如何在中央设备侧执行最常见的蓝牙低能耗任务。在本章中，你将学习到如何使用 Core Bluetooth 框架来在外围设备侧执行最常见的蓝牙低能耗任务。下面的代码示例会帮助你开发应用来在本地设备实现外围角色。特别地，你会学到：

* 开启外围管理者对象
* 在本地外围设备上建立服务和特性
* 发布服务和特性到设备本地数据库
* 广播服务
* 响应已连接的中央设备的读写请求
* 发送更新特性值到已订阅的中央设备

本章的代码例子简单并且抽象，需要适当的修改以合并到真实应用中使用。在本地设备上实现外围角色的更高级的专题（包括小窍门、技巧以及最佳实践），囊括在后面的章节中，[Core Bluetooth Background Processing for iOS Apps](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html#//apple_ref/doc/uid/TP40013257-CH7-SW1) 和 [Best Practices for Setting Up Your Local Device as a Peripheral](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral.html#//apple_ref/doc/uid/TP40013257-CH5-SW1)。

## 开启外围管理者

在本地设备上实现外围角色的第一步是分配并初始化一个外围管理者实例（由 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 对象表示）。通过调用 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 类的 [initWithDelegate:queue:options:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393295-initwithdelegate) 方法来开启外围管理者，像这样：

```
myPeripheralManager =
        [[CBPeripheralManager alloc] initWithDelegate:self queue:nil options:nil];
```

在这个例子中，`self` 被设置成代理以接收任何与外围角色相关的事件。当指定派发队列为 `nil` 时，外围管理者会使用主队列来派发外围角色事件。

当创建外围管理者时，外围管理者会调用它代理对象的 [peripheralManagerDidUpdateState:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393271-peripheralmanagerdidupdatestate) 方法。你必须实现这个代理方法来确保本地外围设备支持蓝牙低能耗并且可用。有关如何实现这个代理方法的信息，请看 [CBPeripheralManagerDelegate Protocol Reference](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate)。

## 建立服务和特性
如[图 1-7](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothOverview/CoreBluetoothOverview.html#//apple_ref/doc/uid/TP40013257-CH2-SW18) 所示，本地外围设备的服务和特性数据库按照树枝状的方式来组织。必须以树枝状的方式来组织以在本地外围设备上建立服务和特性。执行这些任务的第一步是理解服务和特性是如何被识别的。

### 服务和特性由 UUID 识别

外围设备的服务和特性由 128 bit 的蓝牙特有 UUID 识别，在 Core Bluetooth 框架中由 [CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) 对象表示。并非所有的 UUID 都由蓝牙技术联盟（SIG）事先定义，蓝牙技术联盟定义并公布了一组常用的 UUID，并且，为了方便缩短到 16 bit。例如，蓝牙技术联盟预定义了标识心率服务的 16 bit 的 UUID “180D”。这个 UUID 缩短自它的等价 128 bit UUID “0000180D-0000-1000-8000-00805F9B34FB”，这是基于蓝牙 4.0 规范中定义的蓝牙基础 UUID，卷 3 ，F 部分，3.2.1 节。

[CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) 类提供工厂方法使得开发应用时更容易处理过长的 UUID。例如，不是在代码中传递心率服务的 128 bit UUID 字符串表示，而是简单地使用 `UUIDWithString` 方法来用服务事先定义的 16 bit UUID 创建 [CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) 对象，像这样：

```
CBUUID *heartRateServiceUUID = [CBUUID UUIDWithString: @"180D"];
```

当用事先定义的 16 bit UUID 创建 [CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) 对象时，Core Bluetooth 使用蓝牙基础 UUID 预填充 128 bit UUID 剩下的位。

### 为自定义服务和特性创建 UUID

可能会有预定义蓝牙 UUID 无法识别的服务和特性，如果这样，需要生成自己的 128 bit UUID 来识别它。

使用命令行工具 `uuidgen` 来轻松地生成 128 bit 的 UUID。打开终端窗口，然后为需要使用 UUID 识别的每个服务和特性，在命令行中输入 `uuidgen` 以接收唯一的 128 bit 值（以 ASCII 字符串的形式，使用 `-` 号连接），如下所示：

```
$ uuidgen
71DA3FD1-7E10-41C1-B16F-4430B506CDE7
```

然后就可以使用这个 UUID 来创建一个 [CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) 对象，使用 `UUIDWithString` 方法，像这样：

```
CBUUID *myCustomServiceUUID =
        [CBUUID UUIDWithString:@"71DA3FD1-7E10-41C1-B16F-4430B506CDE7"];
```

### 构建服务特性树

在拿到服务和特性的 UUID 后，就可以创建可变的服务和特性并按照上面说的树枝状的形式来组织它们。例如，如果拿到了一个特性的 UUID，可以通过调用 [CBMutableCharacteristic](https://developer.apple.com/documentation/corebluetooth/cbmutablecharacteristic) 类的 [initWithType:properties:value:permissions:](https://developer.apple.com/documentation/corebluetooth/cbmutablecharacteristic/1519073-init) 方法来创建一个可变的特性。

```
myCharacteristic =
        [[CBMutableCharacteristic alloc] initWithType:myCharacteristicUUID
         properties:CBCharacteristicPropertyRead
         value:myValue permissions:CBAttributePermissionsReadable];
```

当创建可变特性时，设置它的属性、值以及权限。设置的属性和权限决定了特性的值是否可读写以及一个已连接的中央设备是否可以订阅。这个例子中，特性的值设置为中央设备可读。有关可变特性的属性和权限支持的范围，请看 [CBMutableCharacteristic Class Reference](https://developer.apple.com/documentation/corebluetooth/cbmutablecharacteristic)。

> **注意**：如果为特性指定一个值，该值会被缓存并且它的属性和权限被设置为可读的。因此，如果想让特性的值可写，或者期望在特性所属的已发布服务的生命期内改变特性值，那么必须该值为 `nil`。遵循这个方式确保该值被动态地对待，并且外围管理者会在收到已连接中央设备的读写请求时请求该值。

由于已经创建了可变特性，接下来就可以创建一个可变服务与之关联。调用 [CBMutableService](https://developer.apple.com/documentation/corebluetooth/cbmutableservice) 类的 [initWithType:primary:](https://developer.apple.com/documentation/corebluetooth/cbmutableservice/1434330-init) 方法，就像这样：

```
myService = [[CBMutableService alloc] initWithType:myServiceUUID primary:YES];
```

在这个例子中，第二个参数设置为 `YES`，表明服务是主要的（与次要相对）。*主服务* 描述设备的主要功能并且可以被其他服务包含（引用）。*次服务* 描述仅仅在引用它的服务的上下文中有意义的服务。例如，一个心率主服务可能要公开来自监听器的心率传感器的心率数据，而次服务可能要公开传感器的电池数据。

在创建服务之后，可以为它关联特性，通过设置服务的特性数组，像这样：

```
myService.characteristics = @[myCharacteristic];
```

## 发布服务和特性

在构建了服务特性树之后，下一步就是把它们发布到设备的服务和特性数据库。使用 Core Bluetooth 框架，执行这个任务很简单。调用 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 类的 [addService:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393255-addservice) 方法，像这样：

```
[myPeripheralManager addService:myService];
```

当调用这个方法来发布服务时，外围管理者会调用它的代理对象的 [peripheralManager:didAddService:error:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393279-peripheralmanager) 方法。如果错误发生导致服务发布失败，实现这个代理方法来访问导致错误发生的原因，如下所示：

```
- (void)peripheralManager:(CBPeripheralManager *)peripheral
            didAddService:(CBService *)service
                    error:(NSError *)error {
    if (error) {
        NSLog(@"Error publishing service: %@", [error localizedDescription]);
    }
    ...
```

> **注意**：在发布服务以及与之关联的特性到外围设备的数据库之后，服务会被缓存而且再也不能修改它。

## 广播服务

当已发布服务和特性到设备的服务特性数据库时，就准备向中央设备（可能正在监听）开始广播它们中的一部分。如下所示，通过调用 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 类的 [startAdvertising:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393252-startadvertising) 方法来广播服务，传入一个包含广播数据的 [NSDictionary](https://developer.apple.com/documentation/foundation/nsdictionary) 实例：

```
[myPeripheralManager startAdvertising:@{ CBAdvertisementDataServiceUUIDsKey :
        @[myFirstService.UUID, mySecondService.UUID] }];
```

在这个例子中，字典中的仅有的键 [CBAdvertisementDataServiceUUIDsKey](https://developer.apple.com/documentation/corebluetooth/cbadvertisementdataserviceuuidskey)，期望的值为一个数组( [NSArray](https://developer.apple.com/documentation/foundation/nsarray) 的一个实例)，这个数组包含想要广播的服务的 UUID（[CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) 对象）。广播数据的字典中可能指定的键详见 [CBCentralManagerDelegate Protocol Reference](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate) 中的 `Advertisement Data Retrieval Keys` 所描述的常量。也就是说，外围管理者对象仅支持两个键 [CBAdvertisementDataLocalNameKey](https://developer.apple.com/documentation/corebluetooth/cbadvertisementdatalocalnamekey) 和 [CBAdvertisementDataServiceUUIDsKey](https://developer.apple.com/documentation/corebluetooth/cbadvertisementdataserviceuuidskey)。

当开始广播本地外围设备的数据时，外围管理者会调用它的代理对象的 [peripheralManagerDidStartAdvertising:error:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393321-peripheralmanagerdidstartadverti) 方法。如果错误发生导致广播服务失败，实现这个代理方法来访问导致错误发生的原因，像这样：

```
- (void)peripheralManagerDidStartAdvertising:(CBPeripheralManager *)peripheral
                                       error:(NSError *)error {
 
    if (error) {
        NSLog(@"Error advertising: %@", [error localizedDescription]);
    }
    ...
```

> **注意**：数据广播是按照 “尽力而为” 的基本原则来做的，因为空间受限并且可能有多个应用同时在广播。更多信息，请看 [CBPeripheralManager Class Reference](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 中的 [startAdvertising:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393252-startadvertising) 方法的讨论。</br></br>
> 当应用处于后台时，广播的行为也会受到影响。这个专题会在下一章 [Core Bluetooth Background Processing for iOS Apps](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html#//apple_ref/doc/uid/TP40013257-CH7-SW1) 讨论。

一旦开始广播数据，远程中央设备就可以发现并发起连接了。

## 响应来自中央设备的读写请求

在连接远程中央设备之后，开始处理来自它们的读写请求。当处理请求时，确保以恰当的方式来响应这些请求。下面的例子描述了如何处理这样的请求。

当已连接的中央设备请求读取特性值时，外围管理者会调用它的代理对象的 [peripheralManager:didReceiveReadReques:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393257-peripheralmanager) 方法，这个代理方法会以 [CBATTRequest](https://developer.apple.com/documentation/corebluetooth/cbattrequest) 对象（包含一组可以用来完成请求的属性）的形式传递请求。

例如，当接收到一个简单的读取特性值的请求时，从代理方法中接收到的 [CBATTRequest](https://developer.apple.com/documentation/corebluetooth/cbattrequest) 对象的属性可以用来确保设备数据库中的特性与远程中央设备在原始读请求中所指定的那个相匹配。实现这个代理方法，像这样：

```
- (void)peripheralManager:(CBPeripheralManager *)peripheral
    didReceiveReadRequest:(CBATTRequest *)request {
 
    if ([request.characteristic.UUID isEqual:myCharacteristic.UUID]) {
        ...
```

如果该特性的 UUID 匹配，下一步就是确保该读请求没有要求读取超出特性值范围的索引位置。如下所示，可以使用 [CBATTRequest](https://developer.apple.com/documentation/corebluetooth/cbattrequest) 对象的 [offset](https://developer.apple.com/documentation/corebluetooth/cbattrequest/1518857-offset) 属性来确保读请求没有试图读取超出范围的值：

```
if (request.offset > myCharacteristic.value.length) {
        [myPeripheralManager respondToRequest:request
            withResult:CBATTErrorInvalidOffset];
        return;
    }
```

假设请求的偏移量已经验证过，现在设置该请求的特性属性的值（默认值为 `nil`），设置为本地外围设备创建的特性的值，考虑读请求的偏移量：

```
request.value = [myCharacteristic.value
        subdataWithRange:NSMakeRange(request.offset,
        myCharacteristic.value.length - request.offset)];
```

在设置了值之后，响应远程中央设备以表明请求成功地完成了。调用 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 类的 [respondToRequest:withResult:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393293-respondtorequest) 方法，回传请求（值已经被更新）和请求的结果，像这样：

```
[myPeripheralManager respondToRequest:request withResult:CBATTErrorSuccess];
```

每一次 [peripheralManager:didReceiveReadRequest:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393257-peripheralmanager) 方法被调用的时候正好调用一次 [respondToRequest:withResult:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393293-respondtorequest) 方法。

> **注意**：如果特性的 UUID 不匹配，或者由于任何其他原因导致读取不能完成，你不用尝试完成该请求。相反，应该立即调用 [respondToRequest:withResult:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393293-respondtorequest) 方法并提供一个表明失败原因的结果。可能指定的结果的清单，请看 [Core Bluetooth Constants Reference](https://developer.apple.com/documentation/corebluetooth/core_bluetooth_constants) 中的 [CBATTError Constants](https://developer.apple.com/documentation/corebluetooth/cbatterror.code) 枚举。

处理来自中央设备的写请求也很简单。当中央设备发送特性的写请求时，外围管理者会调用它的代理对象的 [peripheralManager:didReceiveWriteRequests:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393315-peripheralmanager) 方法。这回，这个代理方法以数组（包含一个或多个 [CBATTRequest](https://developer.apple.com/documentation/corebluetooth/cbattrequest) 对象，每一个代表一个写请求）的形式传递请求。在确定写请求可以满足时，写入特性的值，像这样：

```
myCharacteristic.value = request.value;
```

虽然上面的来例子没有给出，但是当写入特性值时，要保证考虑到请求的偏移量属性。

就像响应读请求一样，每次 [peripheralManager:didReceiveWriteRequests:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393315-peripheralmanager) 代理方法被调用时都正好调用一次 [respondToRequest:withResult:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393293-respondtorequest) 方法。也就是说， [respondToRequest:withResult:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393293-respondtorequest) 方法的第一个参数期望是一个 [CBATTRequest](https://developer.apple.com/documentation/corebluetooth/cbattrequest) 对象，即使从 [peripheralManager:didReceiveWriteRequests:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393315-peripheralmanager) 代理方法中接收到一个包含一个或多个请求对象的数组。应该传入数组中的第一个请求，像这样：

```
[myPeripheralManager respondToRequest:[requests objectAtIndex:0]
        withResult:CBATTErrorSuccess];
```

> **注意**：像对待单个请求那样对待多个请求，如果有任何个别请求不能满足，就不该满足任何一个请求。相反，应该立即调用 [respondToRequest:withResult:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393293-respondtorequest) 方法并提供一个表明失败原因的结果。

## 发送更新特性值到已订阅的中央设备

通常，中央设备会订阅特性值中的一个或多个，如 [Subscribing to a Characteristic’s Value](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW16) 中所描述。在它们订阅的特性值发生改变之后，你有责任通知它们。下面的例子会介绍怎么做。

当中央设备订阅特性值时，外围管理者会调用它的代理对象的 [peripheralManager:central:didSubscribeToCharacteristic:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393261-peripheralmanager) 方法。

```
- (void)peripheralManager:(CBPeripheralManager *)peripheral
                  central:(CBCentral *)central
didSubscribeToCharacteristic:(CBCharacteristic *)characteristic {
 
    NSLog(@"Central subscribed to characteristic %@", characteristic);
    ...
```

使用上面的代理方法作为线索来开始给中央设备发送更新后的值。

紧接着，获取更新后的特性值并调用 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 类的 [updateValue:forCharacteristic:onSubscribedCentrals:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393281-updatevalue) 方法发送给中央设备。

```
NSData *updatedValue = // fetch the characteristic's new value
    BOOL didSendValue = [myPeripheralManager updateValue:updatedValue
        forCharacteristic:characteristic onSubscribedCentrals:nil];
```

当调用这个方法给已订阅的中央设备发送更新后的特性值时，可以给最后一个参数指定想要更新的中央设备。如上面的例子，如果指定为 `nil`，所有已连接、已订阅的中央设备都会被更新（所有已连接但未订阅的中央设备会被忽略）。

[updateValue:forCharacteristic:onSubscribedCentrals:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393281-updatevalue) 方法返回一个布尔值表明更新是否成功。如果用来传输更新值的底层队列已满，该方法会返回 `NO`。当传输队列有可用的空间时，外围管理者会调用它的代理对象的 [peripheralManagerIsReadyToUpdateSubscribers:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393248-peripheralmanagerisready) 方法。然后可以实现这个代理方法重发该值，再次使用 [updateValue:forCharacteristic:onSubscribedCentrals:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393281-updatevalue) 方法。

> **注意**：使用通知发送单一数据包到已订阅中央设备，也就是说，当你更新中央设备时，应该在一个单独的通知中发送全部的更新值，通过调用一次 [updateValue:forCharacteristic:onSubscribedCentrals:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393281-updatevalue) 方法。</br></br>
> 取决于特性值的尺寸，并非所有的数据都通过通知传输。如果发生，这类情形应该在中央设备侧处理，通过调用 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 类的 [readValueForCharacteristic:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518759-readvalue) 方法可以取回全部值。

# iOS 应用的 Core Bluetooth 后台处理

对 iOS 应用来说，知道应用是运行在前台还是后台是很重要的。因为 iOS 设备上的系统资源更有限，所以应用处于后台时肯定会表现得跟处于前台时不一样。对于 iOS 上的后台操作的讨论，请看 [App Programming Guide for iOS](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007072) 中的 [Background Execution](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4)。

默认情况下，如果应用处于后台或者挂起状态，许多常见的 Core Bluetooth 任务都不可用（中央设备侧和外围设备侧）。也就是说，可以声明应用支持 Core Bluetooth 后台执行模式以允许应用被唤醒去处理某些蓝牙相关的事件。即使应用不需要全范围的后台处理支持，仍然可以要求系统在重要事件发生时给出警告。

即使应用支持一个 Core Bluetooth 后台执行模式或两个都支持，它也不能一直运行。例如，在某一时刻，系统需要终止应用以释放内存给当前处于前台的应用使用，这会导致连接（活跃或者未完成的连接）丢失。在 iOS 7 之后，Core Bluetooth 支持保存中央和外围管理者对象的状态信息并在应用启动的时候恢复状态。可以使用这个特性来支持蓝牙设备的长期活动。

## 仅前台运行的应用

正如大部分的 iOS 应用，除非你请求执行特定后台任务的权限，否则应用在进入后台很快就转变到挂起状态。当处于挂起状态，应用就不能执行蓝牙相关的任务，也觉察不到蓝牙相关的事件，直到重新开始进入前台。

在中央设备侧，仅前台运行的应用（没有声明支持 Core Bluetooth 后台执行模式的应用）在处于后台或者挂起状态时不能扫描和发现广播外围设备。在外围设备侧，广播被禁用，尝试访问应用已发布服务的动态特性值的中央设备会收到错误。

取决于使用场景，这个默认行为会以许多方式影响你的应用。例如，想象你正在与外围设备交互数据，然后进入到挂起状态（比如，因为用户切换到其他应用）。如果当应用被挂起时到外围设备的连接丢失，你不会觉察到连接断开直到应用再次进入前台。

### 利用外围设备连接选项

当仅支持前台运行的应用处于挂起状态时，所有出现的蓝牙相关的事件都会被系统排列起来并在应用重新进入前台时传递过去。也就是说，当某些中央角色事件发生时，Core Bluetooth 提供一种警告用户的方式。用户可以使用这些警告来决定特定的事件是否需要把应用带回到前台。

当调用 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 类的 [connectPeripheral:options](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518766-connect) 方法连接到远程外围设备时，通过包含以下外围连接选项中的一个，可以利用这些警告：

* [CBConnectPeripheralOptionNotifyOnConnectionKey](https://developer.apple.com/documentation/corebluetooth/cbconnectperipheraloptionnotifyonconnectionkey)——如果想让系统在连接成功而应用处于挂起状态时显示警告（对于给定外围设备），包含这个键。
* [CBConnectPeripheralOptionNotifyOnDisconnectionKey](https://developer.apple.com/documentation/corebluetooth/cbconnectperipheraloptionnotifyondisconnectionkey)——如果想让系统在连接断开而应用处于挂起状态时显示警告（对于给定外围设备），包含这个键。
* [CBConnectPeripheralOptionNotifyOnNotificationKey](https://developer.apple.com/documentation/corebluetooth/cbconnectperipheraloptionnotifyonnotificationkey)——如果想让系统在收到通知（来自给定外围设备）而应用处于挂起状态时显示警告，包含这个键。

有关蓝牙连接选项的更多信息，详见 [CBCentralManager Class Reference](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 中的 `Peripheral Connection Options` 常量。 

## Core Bluetooth 后台执行模式

如果应用需要在后台执行某些蓝牙相关的任务，必须在[信息属性列表](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/PropertyLists/Introduction/Introduction.html#//apple_ref/doc/uid/10000048i) (`Info.plist`) 文件中声明支持蓝牙后台执行模式。当声明了这个时，系统会唤醒处于挂起状态的应用并允许它处理蓝牙相关的事件。这个支持对于那些与以规律的时间间隔传递数据的蓝牙低能耗设备进行交互的应用很重要，例如心率监听器。

应用有两个可以声明的 Core Bluetooth 后台执行模式，一个用来实现中央角色，另一个用来实现外围角色。如果应用要实现两个角色，可能会声明支持两种后台执行模式。通过在 `Info.plist` 文件中添加 [UIBackgroundModes](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/plist/info/UIBackgroundModes) 键来声明，然后设置该键的值为一个数组，数组包含以下两个字符串：

* `bluetooth-central`——应用使用 Core Bluetooth 框架来与蓝牙低能耗外围设备通信
* `bluetooth-peripheral`——应用使用 Core Bluetooth 框架来共享数据

> **注意**：Xcode 中的属性列表编辑器默认显示人工可读的字符串而不是实际的键名称。要在 `Info.plist` 文件出现的时候显示实际的键名称，右击编辑器窗口，然后在弹出的对话窗口中选择启用 `Show Raw Keys/Values` 条目。

关于如何配置 `Info.plist` 文件的内容，请看 [Xcode Help](https://help.apple.com/xcode)。

### 中央蓝牙后台执行模式

当实现中央角色的应用在 `Info.plist` 文件中加入 [UIBackgroundModes](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/plist/info/UIBackgroundModes) 键（键的值为 `bluetooth-central`）时，Core Bluetooth 框架允许应用在后台执行某些蓝牙相关的任务。当应用在后台时仍然发现和连接外围设备并且探测和交互数据。另外，当 [CBCentralManagerDelegate](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate) 或 [CBPeripheralDelegate](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate) 的代理方法被调用时系统会唤醒应用，允许应用处理重要的中央角色事件（例如，当连接建立和拆除时、当外围设备发送更新特性值时以及当中央管理者状态改变时）。

虽然应用处于后台时可以执行许多蓝牙相关的任务，请记住在应用处于后台时执行扫描外围设备的操作是不同于前台的。特别是当应用处于后台时扫描设备：

* [CBCentralManagerScanOptionAllowDuplicatesKey](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerscanoptionallowduplicateskey) 扫描选项键被忽略，一个广播外围设备的多次发现会被合并到一个单一的发现事件中
* 如果所有扫描外围设备的应用都在后台，那么中央设备扫描广播包的时间间隔会增加。结果是，需要花更长的时间来发现广播外围设备。

这些改变帮助减少无线电的使用，提高 iOS 设备的电池寿命。

### 外围蓝牙后台执行模式

要在后台执行某些外围角色任务，必须在应用的 `Info.plist` 文件中包含 [UIBackgroundModes](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/plist/info/UIBackgroundModes) 键，值为 `bluetooth-peripheral`。当应用的 `Info.plist` 文件中包含这个键值对，系统会唤醒应用去处理读、写和订阅事件。

当应用处于后台状态时，除了允许应用被唤醒以处理来中央设备的读、写以及订阅请求之外，Core Bluetooth 框架还允许应用发广播。也就是说，你应该意识到在应用处于后台时发广播的操作是不同于前台的。特别是当应用处于后台时发广播：

* [CBAdvertisementDataLocalNameKey](https://developer.apple.com/documentation/corebluetooth/cbadvertisementdatalocalnamekey) 广播键会被忽略，并且外围设备的本地名称不会被广播出去。
* [CBAdvertisementDataServiceUUIDsKey](https://developer.apple.com/documentation/corebluetooth/cbadvertisementdataserviceuuidskey) 广播键的值中包含的所有服务 UUID 会被放入特殊的“溢出”区域，它们只能被显式地扫描它们的 iOS 设备所发现
* 如果所有发广播的应用都在后台，那么外围设备发送广播包的频率可能会减少。

## 明智地使用后台执行模式

尽管声明应用支持一个或两个 Core Bluetooth 后台执行模式可能很有必要以满足特定的使用场景，但是你应当负责任地执行后台处理。因为执行许多蓝牙相关的任务需要积极的使用 iOS 设备的机载无线电，同时，无线电的使用反过来对 iOS 设备的电池寿命有不利影响。尝试着去减少后台执行的任务数量。被唤醒以处理蓝牙相关的事件的应用应该尽可能快地处理并结束，以便应用可以再次被挂起。

声明支持 Core Bluetooth 后台执行模式的应用必须遵循几个基本原则：

* 应用应当基于会话并提供界面让用户决定何时开启和关闭蓝牙相关事件的传递
* 在唤醒到来之际，应用有大约 10 秒钟的时间完成任务。最理想的情况是，应用应该尽可能快地完成任务以使自身再次被挂起。花太多时间在后台执行任务的应用会被系统拦下或者杀掉。
* 应用不该把唤醒当成执行无关任务（与应用被系统唤醒的原因不相关的）的机会

有关应用在后台应该如何运转的更普遍的信息，请看 [App Programming Guide for iOS](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007072) 中的 [Being a Responsible Background App](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW8)。

## 在后台执行长期活动

有些应用可能需要使用 Core Bluetooth 在后台执行长期活动。例如，设想一下你正在为 iOS 设备开发一款家庭安全应用，设备与门锁（配备蓝牙低能耗技术的门锁）通信，应用通过与锁的交互来自动地锁门（当用户离开家时）和开门（当用户回到家时），所有的操作都是应用处于后台时完成。当用户离开家时，iOS 设备最终可能会超出锁的范围，导致与锁的连接丢失，这时，应用可以通过调用 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 类的 [connectionPeripheral:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518766-connect) 方法，因为连接请求不超时，所以在用户回到家时 iOS 设备会重新进行连接。

现在设想一下用户离家几天，如果应用在离开家时被系统终止掉，应用将不能在用户回到家时重新连接到门锁，然后用户可能无法打开房门。像这样的应用，能够继续使用 Core Bluetooth 来执行长期活动是很重要的，例如监听活跃的和未完成的连接。

### 状态保存和恢复

因为状态保存和恢复内置于 Core Bluetooth，所以应用可以选择加入这个特性来要求系统保存应用的中央和外围管理者的状态并且为应用继续执行某些蓝牙相关的任务，甚至是应用不再运行。当这些任务中的某个已完成，系统会重启应用到后台中并让应用有机会去恢复它的状态并恰当地处理事件。至于上面所说的家庭安全应用，系统会监听连接请求，并再次重启应用来处理 [centralManager:didConnectPeripheral:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518969-centralmanager) 代理回调（当用户回家并且连接请求已完成）。

Core Bluetooth 为实现了中央角色或外围角色（或者两个都实现）的应用提供状态保存和恢复的支持。当应用实现了中央角色并添加状态保存和恢复的支持，系统会在它要终止应用以释放内存时保存中央管理者的状态（如果应用有多个中央管理者，你可以选择一个让系统进行记录）。特别地，对于一个给定的 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 对象，系统会记录：

* 中央管理者正在扫描的服务（以及当扫描开始时的扫描选项）
* 中央管理者试图连接或者已连接的外围设备
* 中央管理者订阅的特性

实现了外围角色的应用同样可以利用状态保存和恢复。对于 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 对象，系统会记录：

* 外围管理者正在广播的数据
* 外围管理者发布到设备数据库的服务和特性
* 订阅了特性值的中央设备

当应用被系统重启进入到后台（比如，发现了应用要扫描的外围设备），你可以重新实例化应用的中央管理者和外围管理者并恢复它们的状态。下面的部分详细描述了如何在应用中利用状态保存和恢复。

### 支持状态保存和恢复

Core Bluetooth 中的状态保存和恢复是一个可以选择加入的特性并且需要应用的帮助才能工作。在应用中为这个特性加入支持，遵循以下流程：

1. （必须的）当分配和初始化中央管理者和外围管理者对象时加入状态保存和恢复。这一步在`选择加入状态保存和恢复`中有提到。
2. （必须的）在应用被系统重启后重新实例化中央和外围管理者对象。这一步在 [Reinstantiate Your Central and Peripheral Managers](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html#//apple_ref/doc/uid/TP40013257-CH7-SW13) 中有提到。
3. （必须的）实现适当的恢复代理方法。这一步在 [Implement the Appropriate Restoration Delegate Method](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html#//apple_ref/doc/uid/TP40013257-CH7-SW14) 中有提到。
4. （可选的）修正中央和外围管理者的初始化流程。这一步在 [Update Your Initialization Process](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html#//apple_ref/doc/uid/TP40013257-CH7-SW15) 中有提到。

#### 选择加入状态保存和恢复

要加入状态保存和恢复特性，仅需要在分配和初始化中央和外围管理者时提供一个唯一恢复标识符。*恢复标识符* 是应用和 Core Bluetooth 用来识别中央或者外围管理者的一个字符串。字符串的值只对你的代码有意义，但该字符串的出现告诉 Core Bluetooth 被标记的对象需要保存状态。Core Bluetooth 仅保存包含恢复标识符的对象的状态。

例如，如果要为仅使用一个 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 对象来实现中央角色的应用加入状态保存和恢复，那么，当你分配和初始化中央管理者的时候，就指定 [CBCentralManagerOptionRestoreIdentifierKey](https://developer.apple.com/documentation/corebluetooth/cbcentralmanageroptionrestoreidentifierkey) 初始化选项并为中央管理者提供恢复标识符。

```
myCentralManager =
        [[CBCentralManager alloc] initWithDelegate:self queue:nil
         options:@{ CBCentralManagerOptionRestoreIdentifierKey:
         @"myCentralManagerIdentifier" }];
```

虽然上面的例子没有演示这个，但可以用类似的方式来为使用外围管理者的应用加入状态保存和恢复：指定 [CBPeripheralManagerOptionRestoreIdentifierKey](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanageroptionrestoreidentifierkey) 初始化选项并在分配和初始化管理者对象时提供恢复标识符。

> **注意**：因为应用可能有多个 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 和 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 对象的实例，所以要确保每个恢复标识符都是唯一的，以便系统区分不同的外围或者中央管理者对象。

#### 重新实例化中央和外围管理者

当应用被系统重启进入后台，你需要做的第一件事就是使用相同的标识符（第一次创建时所用的恢复标识符）重新实例化适当的中央和外围管理者。如果应用仅有一个中央或外围管理者，并且该管理者在应用的生命期内一直存在，那么这一步你没有别的要做了。

如果应用包含一个以上的中央或者外围管理者（或者使用一个管理者，该管理者并非在应用生命期内一直存在），那么应用需要知道该实例化哪个管理者（当应用被系统重启时）。当实现应用的 [application:didFinishLaunchingWithOptions:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622921-application) 代理方法时，通过使用恰当的启动选项键 [UIApplicationLaunchOptionsBluetoothCentralsKey](https://developer.apple.com/documentation/uikit/uiapplicationlaunchoptionsbluetoothcentralskey) 或 [UIApplicationLaunchOptionsBluetoothPeripheralsKey](https://developer.apple.com/documentation/uikit/uiapplicationlaunchoptionskey/1623116-bluetoothperipherals)， 你可以访问管理者对象的恢复标识符列表（系统在应用终止时保存下来的）。

例如，当应用被系统重启，你可以获取系统为应用保存的所有中央管理者对象的恢复标识符，像这样：

```
- (BOOL)application:(UIApplication *)application
didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
 
    NSArray *centralManagerIdentifiers =
        launchOptions[UIApplicationLaunchOptionsBluetoothCentralsKey];
    ...
```

在拿到恢复标识符列表后，仅仅需要遍历并重新实例化合适的中央管理者对象。

> **注意**：当应用被重启，系统仅仅为执行蓝牙相关任务的中央和外围管理者提供恢复标识符（当应用不再运行）。这些启动选项键在 [UIApplicationDelegate Protocol Reference](https://developer.apple.com/documentation/uikit/uiapplicationdelegate) 中有更多的描述。

#### 实现适当的恢复代理方法

在重新实例化中央和外围管理者后，通过与蓝牙系统进行状态同步来恢复。要把应用带回到系统替它保存的状态（当应用不运行时），必须实现恰当的恢复代理方法。对于中央管理者，实现 [centralManager:willRestoreState:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518819-centralmanager) 代理方法，对于外围管理者，实现 [peripheralManager:willRestoreState:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393317-peripheralmanager) 代理方法。

> **重要**：对于加入 Core Bluetooth 状态保存和恢复特性的应用，当应用被重启进入到后台以完成蓝牙相关的任务时，这两个（ [centralManager:willRestoreState:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518819-centralmanager) 和 [peripheralManager:willRestoreState:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393317-peripheralmanager) ）是*首先*被调用的方法。对于没有加入状态恢复和保存特性的应用（或者在启动时没有需要恢复的东西），那么首先调用的两个方法是 [centralManagerDidUpdateState:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518888-centralmanagerdidupdatestate) 和 [peripheralManagerDidUpdateState:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393271-peripheralmanagerdidupdatestate)。

在上面的两个代理方法中，最后一个参数（是一个字典）包含应用终止时保存下来的管理者信息。可用的字典键清单，请看 [CBCentralManagerDelegate Protocol Reference](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate) 中的 `中央管理者状态恢复选项` 常量和 [CBPeripheralManagerDelegate Protocol Reference](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate) 中的 `外围管理者状态恢复选项` 常量。

要恢复 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 对象的状态，使用 [centralManager:willRestoreState:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518819-centralmanager) 代理方法中提供的字典的键。例如，当应用终止时，如果中央管理者对象还有活跃的或者未完成的连接，那么系统会继续为应用监听连接。如下所示，可以使用 [CBCentralManagerRestoredStatePeripheralsKey](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerrestoredstateperipheralskey) 字典键来获取中央管理者已连接或正在尝试连接的所有外围设备（由 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 对象表示）的列表。

```
- (void)centralManager:(CBCentralManager *)central
      willRestoreState:(NSDictionary *)state {
 
    NSArray *peripherals =
        state[CBCentralManagerRestoredStatePeripheralsKey];
    ...
```

在上面的例子中，如何处理恢复的外围设备列表取决于使用场景。例如，如果应用要保存中央管理者发现的外围设备的一个列表，那么可能会把恢复的外围设备加入到该列表中以便引用它们。如 [Connecting to a Peripheral Device After You’ve Discovered It](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW4) 所描述，确保设置外围管理者的代理以保证收到适当的回调。

通过使用 [peripheralManager:willRestoreState:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanagerdelegate/1393317-peripheralmanager) 代理方法中字典的键，你可以以类似的方式来恢复 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 对象的状态。

#### 修正初始化流程

在完成了前面的三个必须步骤之后，你可能想看看如何修正中央和外围管理者的初始化流程。尽管这是一个可选的步骤，但是也很重要，因为可以确保应用运行平滑。例如，当应用在探索外围设备的数据的中途被终止。当应用恢复了外围的状态，它并不知道当它被终止时它在发现外围设备的过程中走了多远。你会想要确保开始的点正是在发现过程中离开的那个点。

例如，当在 [centralManagerDidUpdateState:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518888-centralmanagerdidupdatestate) 代理方法中初始化应用，你可以查明是否成功地从恢复的外围设备发现了特定的服务（在应用被终止前），像这样：

```
NSUInteger serviceUUIDIndex =
        [peripheral.services indexOfObjectPassingTest:^BOOL(CBService *obj,
        NSUInteger index, BOOL *stop) {
            return [obj.UUID isEqual:myServiceUUIDString];
        }];
 
    if (serviceUUIDIndex == NSNotFound) {
        [peripheral discoverServices:@[myServiceUUIDString]];
        ...
```

如上例所示，如果系统在应用完成发现服务之前终止了应用，那么在那时通过调用    [discoverServices:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518706-discoverservices) 来开始探索恢复的外围设备的数据。如果应用成功地发现服务，那么你紧接着就可以检查看看特性是否已被发现（还有你是否已经订阅了特性）。通过以这样的方式修正初始化流程，可以确保在正确的时间调用了正确的方法。

# 与远程外围设备交互的最佳实践

Core Bluetooth 框架使得许多中央设备侧的事务对应用透明。也就是，应用能够支配、有责任实现中央角色的大多数方面，例如发现和连接设备以及特索并与远程外围设备的数据。本章提供以负责的方式来利用这种层面的控制的指导和最佳实践，尤其是在 iOS 设备上开发应用。

## 留心无线电的使用和电量消耗

当开发与蓝牙低能耗设备交互的应用时，请记得蓝牙低能耗技术通信共享设备的无线电以在空中传递信号。其他形式的无线电通信可能也需要使用设备的无线电，例如 Wi-Fi，传统蓝牙，甚至是其它使用蓝牙低能耗的应用，因此，开发应用应该减少使用无线电。

当为 iOS 设备开发应用时，减少无线电的使用尤其重要，因为无线电的使用对设备电池的寿命有不良影响。下面的指导会帮助你成为设备无线电的良好公民。结果是，应用会表现更好而电池寿命会更长。

### 需要时再去扫描设备

当调用 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 类的 [scanForPeripheralsWithServices:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518986-scanforperipheralswithservices) 方法去发现远程外围设备（正在广播服务）时，中央设备会使用它的无线电去监听广播设备直到显式地让它停下。

除非需要发现更多的设备，否则在找到想要连接的设备后应该停止扫描其它设备。
使用 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 类的 [stopScan](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518984-stopscan) 方法去停止扫描其它设备，如 [Connecting to a Peripheral Device After You’ve Discovered It](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW4) 所说。

### 只在必要的时候指定 CBCentralManagerScanOptionAllowDuplicateKey 选项

远程外围设备每秒可能发出很多个广播数据包来向正在监听的中央设备表明它的存在。当使用 [scanForPeripheralsWithServices:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518986-scanforperipheralswithservices) 方法来扫描设备时，方法的默认行为是合并广播设备的多个发现到一个单一发现事件中，也就是，中央管理者会为每一个它发现的新外围设备调用它的代理对象的 [centralManager:didDiscoverPeripheral:advertisementData:RSSI:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518937-centralmanager) 方法，不管它收到多少广播数据包。当已发现的外围设备的广播数据发生改变时，中央管理者也会调用这个代理方法。

如果想要改变默认的行为，当调用 [scanForPeripheralsWithServices:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518986-scanforperipheralswithservices) 方法时可以指定 [CBCentralManagerScanOptionAllowDuplicatesKey](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerscanoptionallowduplicateskey) 常量作为扫描选项。这样的话，每一次中央设备收到来自外围设备的数据时都会生成一个发现事件。关闭默认行为在某些使用场景下可能很有用，例如基于外围设备的邻近度来初始化一条到外围设备的连接（使用外围设备接受信号强度指示器（RSSI）的值）。即便如此，请记住指定这个扫描选项对电池寿命以及应用的性能有不良影响。因此，只在必要的时候再去指定这个扫描选项以满足特定的使用场景。

### 明智地探索外围设备的数据

外围设备的服务和特性数量可能比你感兴趣的要多得多。发现外围设备的所有服务及特性对电池的寿命和应用的性能有不良应用。因此，应该只寻找应用需要的服务以及与之关联的特性。

例如，设想你正在连接到一台有许多可用服务的外围设备，但应用只需要服务中的两个。你可以只寻找这两个服务，通过在 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 类的 [discoverServices:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518706-discoverservices) 方法传入一个服务 UUID（由 [CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) 对象表示） 数组，像这样：

```
[peripheral discoverServices:@[firstServiceUUID, secondServiceUUID]];
```

在找到这两个感兴趣的服务之后，可以以类似的方式寻找这些服务中你感兴趣的特性。如前，只在 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 类的 [discoverCharacteristics:forService:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518797-discovercharacteristics) 方法中为每个服务传入一个 UUID（用来识别你所感兴趣的特性） 数组。


### 订阅频繁改变的特性值

如 [Retrieving the Value of a Characteristic](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW7) 所说，有两种方式可以取回特性的值：

* 每一次需要值的时候，可以通过调用 [readValueForCharacteristic:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518759-readvalue) 方法显式地拉取特性值
* 通过调用一次 [setNotifyValue:forCharacteristic:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518949-setnotifyvalue) 方法来订阅特性值，以便在值发生改变时接收到来自外围设备的通知

可能的话，最好是订阅特性的值，尤其是那些经常改变的特性值。如何订阅特性值的例子，请看 [Subscribing to a Characteristic’s Value](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW16)。

### 拿到所有所需数据后断开连接

当连接不再需要时，通过断开到外围设备的连接可以帮助减少应用的无线电的使用。在以下的两个场景中你应该断开到外围设备的连接：

* 所有订阅的特性值已经停止发送通知。（可以通过访问特性的 [isNotifying](https://developer.apple.com/documentation/corebluetooth/cbcharacteristic/1519057-isnotifying) 属性来判断特性的值是否在发送通知）
* 拿到了所有所需数据

在这两种情况下，取消所有订阅然后断开到外围设备的连接。通过调用 [setNotifyValue:forCharacteristic:](https://developer.apple.com/documentation/corebluetooth/cbperipheral/1518949-setnotifyvalue) 方法并设置第一个参数值为 `NO` 来取消特性值的订阅。通过调用 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 类的 [cancelPeripheralConnection:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518952-cancelperipheralconnection) 方法可以取消到外围设备的连接，像这样：

```
[myCentralManager cancelPeripheralConnection:peripheral];
```

> **注意**：[cancelPeripheralConnection:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518952-cancelperipheralconnection) 方法是非阻塞的，任何仍然挂在你正在尝试断开的外围设备的 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 命令可能会或不会完成执行。因为其他应用可能仍然连着外围设备，所以取消一个本地连接并不能保证底层的物理连接马上断开。不管怎么说，从你应用的角度来看，外围设备被认为是断开的，然后中央管理者对象会调用它的代理对象的 [centralManager:didDisconnectPeripheral:error:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518791-centralmanager) 方法。

## 重新连接到外围设备

使用 Core Bluetooth 框架，有三种可以重连外围设备的方式：

* 使用 [retrievePeripheralsWithIdentifiers:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1519127-retrieveperipherals) 方法取回已知外围设备（以往发现过的或者连接过的外围设备）的列表。如果正在寻找的外围设备在列表中，尝试去连接它。这个重连选项在 [Retrieving a List of Known Peripherals](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW10) 中有讲到。
* 使用 [retrieveConnectedPeripheralsWithServices:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518924-retrieveconnectedperipherals) 方法取回当前正连接到系统的外围设备的列表。如果正在寻找的外围设备在列表中，将它本地连接到应用。这个重连选项在 [Retrieving a List of Connected Peripherals](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW11) 中有讲到。
* 使用 [scanForPeripheralWithServices:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518986-scanforperipheralswithservices) 方法来扫描和发现外围设备。如果找到，就进行连接。这些步骤在 [Discovering Peripheral Devices That Are Advertising](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW3) 和 [Connecting to a Peripheral Device After You’ve Discovered It](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW4) 中有描述。

取决于使用场景，你可能不想每次要重连的时候都去扫描并发现同样的外围设备，相反，你可能想要优先使用其它的选项去试着重连。如图 5-1 所示，一个可能的重连工作流可能是按照上面列出的顺序来尝试每一个选项。

**图 5-1** 重连工作流示例

![重连工作流示例](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/ReconnectingToAPeripheral_2x.png)

> **注意**：决定要尝试的重连选项数量以及这么做的顺序，可能随着应用试图要满足的使用场景变化。例如，你可能决定不使用第一个连接选项，或者并行使用头两个选项。

### 取回已知的外围设备的列表

第一次发现外围设备时，系统会生成一个标识符（一个 UUID，由 [NSUUID](https://developer.apple.com/documentation/foundation/nsuuid) 对象表示）来识别外围设备。然后你可以保存这个标识符（例如，使用 [NSUserDefaults](https://developer.apple.com/documentation/foundation/userdefaults)），之后可以用它来连接到外围设备（使用 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 类的 [retrievePeripheralsWithIdentifiers:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1519127-retrieveperipherals) 方法）。下面讲述了使用这个方法来重连先前已经连接过的外围设备的一种方式。

当应用启动时，调用 [retrievePeripheralsWithIdentifiers:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1519127-retrieveperipherals) 方法并传入一个标识符数组（包含先前发现过并连接过的外围设备的标识符），像这样：

```
knownPeripherals =
        [myCentralManager retrievePeripheralsWithIdentifiers:savedIdentifiers];
```

中央管理者试着把你提供的标识符与先前找到过的外围设备的标识符相比对，并且返回一个包含 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 对象的数组。如果比对发现没有，会返回一个空数组然后你应该尝试另外两个重连选项。如果数组不为空，提供 UI 让用户选择重连哪个外围设备。

当用户选择了一个外围设备，试着通过调用 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 类的 [connectPeripheral:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518766-connectperipheral) 方法来连接该设备。如果设备可以连接上，中央管理者回调用它的代理对象的 [centralManager:didConnectPeripheral:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518969-centralmanager) 方法，然后外围设备就成功地重连上了。

> **注意**：一个外围设备可能由于某些原因不能连接上，例如，设备可能不在中央设备的附近。另外，某些蓝牙低能耗设备使用随机的设备地址（周期性地改变）。因此，即使设备就在附近，从上次被系统发现到现在，设备的地址可能已经改变，在这种情况下，试图连接的 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 对象与实际的外围设备不相符。如果因为地址变化而不能连接到外围设备，那么必须通过使用 [scanForPeripheralsWithServices:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518986-scanforperipheralswithservices) 方法来发现它。</br></br>
> 有关随机设备地址的更多信息，请看蓝牙 4.0 规范，卷 3，C 部分，10.8 节以及 [Bluetooth Accessory Design Guidelines for Apple Products](https://developer.apple.com/hardwaredrivers/BluetoothDesignGuidelines.pdf)。

### 取回已连接的外围设备的列表

另外一种连接到外围设备的方式是：通过检查正在寻找的外围设备是否已经与系统相连（例如，被其它应用连接了）。通过调用 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 类的 [retrieveConnectedPeripheralsWithServices:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518924-retrieveconnectedperipherals) 方法可以这么做，这会返回一个包含 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) 对象（表示当前正在与系统相连的外围设备）的数组。

因为可能存在多个正在与系统相连的外围设备，你可以传入一个包含 [CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) （表示服务的 UUID） 的数组，以取回当前正在与系统相连的并且包含指定 UUID 所标识的服务的外围设备。如果当前没有设备与系统相连，数组会为空，然后你应该尝试其它两个重连选项。如果数组不为空，提供 UI 让用户选择重连哪个外围设备。

假设用户找到并选择了期望的外围设备，通过调用 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager) 类的 [connectPeripheral:options:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/1518766-connectperipheral) 方法，将它本地连接到应用（即使设备已经连接到系统，你仍然必须将它本地连接到应用以开始探索并与之交互）。当本地连接建立后，中央管理者会调用它的代理对象的 [centralManager:didConnectPeripheral:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518969-centralmanager) 方法，然后外围设备就成功地重连上了。

# 把本地设备当作外围设备来使用的最佳实践

正如许多中央设备侧的事务，Core Bluetooth 框架给你控制实现外围角色的大多数方面。这个章节提供以负责任的方式去利用这种层级的控制的指导和最佳实践。

## 广播方面的考虑

广播外围设备数据是建立本地设备以实现外围角色的一个重要部分。下面的章节帮助你以合适的方式实现外围角色。

### 考虑广播数据的限制

通过往 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 类的 [startAdvertising:](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393252-startadvertising) 方法传入一个广播数据的字典来广播外围设备数据，就像 [Advertising Your Services](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonPeripheralRoleTasks/PerformingCommonPeripheralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH4-SW5) 所讲。当创建广播字典时，请记住广播的内容及数量都有限制。

虽然广播数据包通常可以保存各种各样关于外围设备的信息，然而你可能仅仅广播设备的本地名称以及服务的 UUID。因此，当创建广播字典时，或许会只指定以下两个键： [CBAdvertisementDataLocalNameKey](https://developer.apple.com/documentation/corebluetooth/cbadvertisementdatalocalnamekey) 和 [CBAdvertisementDataServiceUUIDsKey](https://developer.apple.com/documentation/corebluetooth/cbadvertisementdataserviceuuidskey)。如果指定了其它键会收到错误。

当广播数据时，能够使用的空间大小也有限制。当应用在前台，任何两种支持的广播数据键的组合的最初的广播数据包中，能够使用多达 28 个字节的空间。如果这个空间被用完了，在扫描响应中还有额外的 10 个字节的空间大小（仅供本地名称使用）。任何与分配空间不适合的服务 UUID，会被添加到特别的 “溢出” 区域，它们只能被显式扫描它们的 iOS 设备发现。当应用处于后台，本地名称不会被广播并且所有的服务 UUID 会被放入溢出区域。

> **注意**：这些尺寸不包括 2 个字节的头部信息（新数据类型需要的头部信息）。确切的广播和响应数据格式在蓝牙 4.0 规范中有定义，卷 3，C 部分，11 小节。

为不超出空间限制，限制向那些标识你的主要服务的用户广播服务 UUID。

### 仅在需要时广播数据

由于广播外围设备的数据使用本地设备的无线电（使用设备的电池），因此，仅在你想让其它设备连接时再去广播。一旦连接上，这些设备就可以直接探索并交互外围设备的数据，不需要任何的广播数据包。因此，为了减少无线电的使用，并提高应用的性能以及保存设备的电量，当没有必要再去利用蓝牙低能耗事务时，停止广播。要在本地外围设备上停止广播，只需要调用 [CBPeripheralManager](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager) 类的 [stopAdvertising](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393275-stopadvertising) 方法，像这样：

```
[myPeripheralManager stopAdvertising];
```

#### 让用户决定何时广播

要知道，何时广播通常只有用户知道。例如，当知道周围没有蓝牙低能耗设备时，还让应用发广播是没有意义的。因为应用通常不知道周围有其它的什么设备，所以请在应用中提供图形界面（UI）的方式来让用户决定何时发广播。

## 配置特性

当创建可变特性时，设置它的属性、值以及权限。这些设置决定了中央设备如何访问特性值以及如何与特性值交互。虽然你可能根据应用的需要决定有区别地配置特性的属性和权限，但下面的章节还是提供了一些指导，当你想要执行以下两个任务时：

* 允许已连接的中央设备订阅你的特性
* 保护敏感特性值以免被未配对的中央设备访问

### 配置特性以支持通知

如 [Subscribe to Characteristic Values That Change Often](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW7) 中所讲，推荐中央设备订阅频繁改变的特性值（一个远程外围设备的服务的特性值）。如果可能，鼓励这种做法（通过允许已连接的中央设备订阅你的特性值）。

当创建可变特性时，通过给特性的属性设置 [CBCharacteristicPropertyNotify](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicproperties/1518976-notify) 常量来支持订阅，像这样：

```
myCharacteristic = [[CBMutableCharacteristic alloc]
        initWithType:myCharacteristicUUID
        properties:CBCharacteristicPropertyRead | CBCharacteristicPropertyNotify
        value:nil permissions:CBAttributePermissionsReadable];
```

在这个例子中，特性的值是可读的，并且可以被已连接的中央设备订阅。

### 需要配对连接以访问敏感数据

取决于使用场景，你可能想要发布服务，该服务有一个或者多个特性值需要确保安全。例如，设想你要发布一个社交媒体资料服务。这个服务的有些特性的值代表会员的资料信息，例如姓名、邮箱地址。很有可能，你只想让受信任的设备取回会员的邮件地址。

通过设置适当的特性属性和权限可以确保只有受信任的设备才能访问敏感特性值。继续上面的例子，为了让只有受信任的设备才能取回会员的邮件地址，像这样设置特性的属性和权限：

```
emailCharacteristic = [[CBMutableCharacteristic alloc]
        initWithType:emailCharacteristicUUID
        properties:CBCharacteristicPropertyRead
        | CBCharacteristicPropertyNotifyEncryptionRequired
        value:nil permissions:CBAttributePermissionsReadEncryptionRequired];
```

这个例子中，特性被配置以让只有受信任的设备读取或订阅它的值。当一个已连接的远程中央设备试图去读取或订阅这个特性的值，Core Bluetooth 会尝试着把本地的外围设备与中央设备进行配对，以建立一条安全连接。

例如，如果中央设备和外围设备都是 iOS 设备，那么两个设备都会收到一个弹框警告表明有其它设备想要配对。在中央设备上的弹框包含一个密码，你必须在外围设备上的文本框中输入这个密码才能完成配对过程。

在配对过程完成之后，外围设备把配对的中央设备当作信任设备，并允许该中央设备访问它的加密特性值。

# 修订历史

这个表格叙述了 Core Bluetooth Programming Guide 的修改。

|日期|备注|
|:----|:----|
|2013-09-18|为 iOS 7 和 OS X v10.9 更新有关在后台执行长期活动以及获取外围设备的新方法的信息|
|2013-08-08|新文档，讲述如何使用 Core Bluetooth 框架来开发与蓝牙低能耗技术交互的应用|
