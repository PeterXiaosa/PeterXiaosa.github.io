---
layout: wiki
title: Android Notices
categories: Android
description: Android 踩坑专集
keywords: Android
---

* Android 9.0 及之前的版本和 Android 10.0 中 `onNewIntent()` 和 `onRestart()`执行顺序不一致。  

* adb shell am start -W packageName/Activity （activity为绝对路径）  可用来启动应用，并且查看冷启动耗时