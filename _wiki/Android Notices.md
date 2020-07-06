---
layout: wiki
title: Android Notices
categories: Android
description: Android 踩坑专集
keywords: Android
---

* Android 9.0 及之前的版本和 Android 10.0 中 `onNewIntent()` 和 `onRestart()`执行顺序不一致。  

* adb shell am start -W packageName/Activity （activity为绝对路径）  可用来启动应用，并且查看冷启动耗时

* ContentProvider 可用来初始化。例如LeakCanary 2.0中，在ContentProvider的`onCreate()`方法中执行初始化代码。这样就可以避免在Application的`onCreate()`中初始化，简化接入流程。ContentProvider的启动流程会在Application的`onCreate()`之前。因为ActivityThread中的`handleBindApplication()`中会去解析AndroidManifest文件，并且调用`installContentProvider()`初始化ContentProvider,然后调用ContentProvider的`attachInfo()`函数，从而执行ContentProvider的`onCreate()`函数。`handleBindApplication()`中整体的流程是会先创建Application,然后执行Application的`attachBaseContext()`方法，接着会执行`installContentProvider()`也即会执行到ContentProvider的`onCreate()`,然后再执行Application的`onCreate()`。就是因为这样的顺序，所以才可以将在Application的`onCreate()`中初始化的代码放置在ContentProvider的`onCreate()`去进行执行。优点是例如SDK接入的时候不需要手动接入，缺点是没有办法去进行相应的按需加载或者延时加载，无法控制应用启动时间。