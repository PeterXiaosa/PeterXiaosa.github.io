---
layout: post
title: 起舞者的艺术-Choreographer
categories: [Android, UI渲染, Choreographer]
description: Android中UI渲染的简单介绍及Choreographer的工作原理。
keywords: Android, UI渲染, Triple Buffer, VSync,Choreographer, Project Butter
---

之前看Matrix源码，看到了在Matrix中的一个`UIThreadMonitor`中涉及到了对 IdleHandler以及 Choreographer 中的反射，没有看太懂。所以后又返回了去看 [IdleHandler](https://peterxiaosa.github.io/2020/07/14/IdleHandler%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90/)，Android闲时处理机制。了解了之后接着又来看与本文有关的`Choreographer`机制，而在了解`Choreographer`之前，我们需要了解一个名词 `Project Butter`。 它是Google在2012年的 I/O 大会上宣布的黄油计划，目的是为了解决Android UI流畅性的问题。Project Butter引入了三个核心的元素，VSYNC、Triple Buffer 和 Choreographer。接下来，我们来分别介绍这三个核心元素。

### VSYNC 

VSYNC，Vertical Synchronization，垂直同步。了解它之前，我们需要首先了解GPU的帧速率和显示器的刷新速率。

* 刷新速率  

表示屏幕每秒刷新的次数，Android设备中一般为60hz,即每秒刷新60次。

* 帧速率

表示GPU每秒绘制的帧数。在电影行业中，电影的刷新频率一般为24fps。这些值是根据人眼的视觉暂留时间来计算得出的。只要fps足够高，由于视觉暂留才会让人感觉物体在运动，从而产生动画。而Android设备中则采用了更为流畅的60fps。

了解了这两个概念之后，我们还需要知道一种现象：屏幕撕裂。

#### 屏幕撕裂
CPU


