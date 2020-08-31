---
layout: wiki
title: Android Knowledage
categories: Android Java
description: Android、Java 知识点的收集
keywords: Android、Java
---

身为一个Android开发工程师，有些知识点还是很有必要掌握的。


## Android

* TCP和UDP
* [CrashHandler的实现](https://peterxiaosa.github.io/2020/08/31/%E5%85%B3%E4%BA%8ECrashHandler%E4%B8%8EBugly%E8%81%94%E5%90%88%E4%BD%BF%E7%94%A8%E5%86%B2%E7%AA%81%E7%9A%84%E9%97%AE%E9%A2%98/)
* [Android各版本的变化](https://peterxiaosa.github.io/2019/09/16/Android%E7%89%88%E6%9C%AC%E7%89%B9%E6%80%A7/)
* Binder的理解及原理。
* [Android消息机制](https://peterxiaosa.github.io/2019/05/29/Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6/)(Looper，MessageQueue，Handler工作原理，如何手写looper，即阻塞消息队列实现)
* [Init进程，Zygote进程，SystemServer进程及Launcher启动流程。](https://peterxiaosa.github.io/2019/09/09/Android%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B/)根Activity启动流程，四大组件自动流程。
* ActivityThread，AMS，WMS的工作原理。
* [IntentService、HandlerThread实现原理](https://github.com/PeterXiaosa/practiceProject/tree/master/app/src/main/java/com/peter/practiceproject/studyitem/handlerthread)。
* 框架的源码。([Eventbus](https://peterxiaosa.github.io/2019/06/06/Eventbus%E8%BF%90%E8%A1%8C%E6%B5%81%E7%A8%8B%E6%B5%85%E6%9E%90/)、OkHttp、Glide)
* 设计模式（单例模式(懒汉，饿汉)，DCL，静态内部类创建）、Builder模式、观察者模式、装饰模式、策略模式、简单工厂模式、抽象工厂模式、代理模式（动态代理）
* 从输入一个URL到看到一个页面的过程。
* Android的性能优化有哪些方法
* Android类加载器，及PathClassloader和DexClassloader有什么用。
* View的事件分发机制。
* 架构设计(MVP和MVC使用及区别)
* ANR原理及如何查看。
* 内存泄露及发生的情况。
* LRU原理
* ListView及Recyclerview上滑卡顿原因。
* Android的线程通信。

## Java

* 动态代理原理。
* [HashMap实现原理](https://peterxiaosa.github.io/2020/04/21/HashMap%E7%9A%84%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/)。（实现结构，hash和rehash，多线程下死锁，红黑树）自定义对象是否可以当做key。及满足条件。(equals及hashcode)
* 由HashMap是否有序，引出有序的TreeMap和LinkedHashMap。及多线程下引出的CourrentHashMap。各实现原理。
* 类的加载机制，Java三种类加载器。及和Android类加载器相比较。
* JVM内存区域，垃圾回收机制(标记计数算法，可达性分析算法)。GC算法（标记-清除算法，复制算法，标记-整理算法）及各运用场景和为什么。Java中的四种引用。
* Synchronized和Volatile区别。
* [ArrayList](https://peterxiaosa.github.io/2020/07/31/ArrayList%E7%9A%84%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E5%8F%8A%E4%BD%BF%E7%94%A8%E4%BB%8B%E7%BB%8D/)和[LinkedList](https://peterxiaosa.github.io/2020/08/02/%E7%8E%A9%E8%BD%ACLinkedList-%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/)区别及源码层板分析下的实现原理。
* [线程池底层实现原理](https://peterxiaosa.github.io/2019/11/03/Java%E7%BA%BF%E7%A8%8B%E6%B1%A0/)