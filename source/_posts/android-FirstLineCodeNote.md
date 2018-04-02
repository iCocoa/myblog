---
title: 《第一行代码》阅读笔记
date: 2017-10-14 11:59:26
tags: Android
---
最近为了学习 Android，找从事 Android 开发的朋友推荐些书，最后他推荐了《第一行代码》（第 2 版）和《Android 开发艺术探索》两本书。本文是在阅读了《第一行代码》之后所做的笔记，主要记录 Android 平台上的一些比较有趣的特性以及它和 iOS 的不同之处。

## Android 全貌

* 2008 年 9 月，Google 正式发布 Android 1.0 系统
* 2014 年 Google I/O 大会上发布号称史上版本改动最大的 Android 5.0 系统，这版本使用 ART 运行环境替换 Dalvik 虚拟机，同时推出 Wear、Auto、TV 系统
* 2016 年 Google I/O 大会推出 Android 7.0，加入多窗口模式

### Android 系统架构

Android 系统架构分为四层：Linux 内核层、系统运行库层、应用架构层、应用层。

**Linux 内核层**：主要包含一些硬件的底层驱动。

**系统运行库层**：包含 C/C++ 的底层支持库，例如：支持 3D 绘图的 OpenGL|ES 库、浏览器内核 Webkit 库和 SQLite 数据库支持库。另外还包含 Android 运行时库。

**应用架构层**：包含构建应用程序用到的 API，开发人员主要使用这层提供的 API 来构建应用。

**应用层**：包含手机上安装的应用，联系人、短信等。

![Android 系统架构](https://en.wikipedia.org/wiki/File:Android-System-Architecture.svg)

Android 系统为开发人员提供了：

* 四大组件，活动（Activity）、服务（Service）、广播接收器（Broadcast Receiver） 和内容提供器（Content Provider）
* 系统控件
* SQLite 数据库
* 多媒体，音乐、视频、图片、拍照、闹铃等
* 地理位置定位

### Android 开发环境及工具

`环境：JDK + Android SDK`

`工具：Eclipse+ADT 或者 Android Studio`

在 Android Studio 中，使用 Gradle 来构建 Android 项目，笔者在 Mac 上安装 Android Studio 工具时联网安装 Gradle 不成功，最后，自己手动下载后导入到工具中。

后来尝试过在 Mac 上使用 Eclipse+ADT 的工具集，但是下载的 ADT 导入不成功，最后放弃使用 Eclipse。

### Android 工程

Android 系统通过包名来区分不同的应用程序，需要确保报名的唯一性。

Android 的日志打印没有使用 `System.out`，而是重新设计了一个工具类 `android.util.log`，该类日志打印分级比较细，分为 verbose、debug、info、warn、error 5 个等级，同时包含打印时间、过滤等功能。

* gradle 相关文件，通过修改这些 gradle 配置文件可以编译出不同的安装包
* 代码混淆规则文件 `proguard-rules.pro`，指定代码混淆规则，增加破解安装包的人阅读代码的难度
* 逻辑与视图分离的设计，布局文件和 Activity 文件分开


## Android UI 界面

Android UI 界面的编写可以使用可视化编辑器或者使用 XML 代码。

### 常用控件

* TextView 
	
	用来显示一段文本。
	
* Button 

	* 点击事件通过注册监听器来处理
	* 系统默认对 Button 中的所有英文字母自动进行大写转换，可以使用 `android:textAllCaps="false"` 来禁用这一默认属性
	
* EditText
	
	文本输入框。
	
* ImageView
	
	用来显示图片。`Nine-Patch` 图片可以指定图片拉伸区域，通过 sdk 中的 `draw8patch.bat` 文件可以生成 Nine-Patch 文件。
	
* ProgressBar
	
	进度条。有圆形进度条和水平进度条两种，可以通过样式来指定。
	
* AlertDialog
	
	确认的对话框。用来提示非常重要的内容或者警告信息，让用户再次确认。会在界面中置顶，并屏蔽其它控件的交互能力。
	
* ProgressDialog
	
	会在对话框中显示一个进度条，用来提示用户等待耗时操作。会在界面中置顶，并屏蔽其它控件的交互能力。

* ListView
	
	列表视图控件，当界面上条目较多时使用。（类似 iOS UITableView）
	
* RecycleView
	
	列表视图控件，增强版本的 ListView，弥补了 ListView 的不足之处。 （类似 iOS UICollectionView）。支持横向滚动和瀑布流布局。

列表视图控件的数据都是通过 `Adapter` 类提供的，该类相当于 MVC 中 ViewModel，Android 通过这个类把这一模式固定下来。ListView 条目的点击事件是通过 ListView 的注册监听器来处理的，没有细化到条目中的子控件，RecycleView 把点击事件处理归到条目中的子控件来处理。

* Fragment

	碎片（Fragment）是一种可以嵌入在活动当中的 UI 片段，使用碎片可以充分利用大屏幕设备的屏幕空间。碎片有自己的生命周期，可以在程序运行时动态地添加到活动中，通过指定 `最小宽度限定符` 指定屏幕宽度最小值，以这个值为临界点，大于和小于这个值的设备分别加载不同的布局。

### UI 布局

Android 中包含 4 种布局： 线性布局（LinearLayout）、相对布局(RelativeLayout)、帧布局(FrameLayout)、百分比布局。布局中放置控件还可以嵌套布局。

* LinearLayout：将包含的控件在线性方向上依次排列。
	
	`android:layout_weight` 属性允许按比例的方式来指定控件的大小（单位 dp）。系统会把 LinearLayout 下所有控件指定的 layout\_weight 值相加，得到一个总值，每个空间所占的大小比例就是该控件的 layout\_weight 的值除以刚才算出的总值。 
	
* RelativeLayout：以相对定位的方式来排列控件。
* 百分比布局：直接指定控件在布局中所占的百分比。包含 PercentFrameLayout 和 PercentRelativeLayout。

## 四大组件

### 活动






### 广播接收器

### 服务

### 内容提供器





