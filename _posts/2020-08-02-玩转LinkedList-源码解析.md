---
layout: post
title: 玩转LinkedList-源码解析
categories: [Java,LinkedList]
description: 对LinkedList的源码分析和使用介绍，从继承，接口，类变量，使用方法，线程安全方面介绍理解。
keywords: LinkedList, Deque, Queue, Java, Stack
---

上一节我们介绍了 Java 中常见的线性集合 Arraylist， 这一节我们再来介绍一下 Java 中另外一种使用频繁的数据结构 Linkedlist。它和 ArrayList 不同，ArrayList 采用的是内部数组，而它内部采用的却是双向链表。接下来，我们就从源码的角度来认识了解一下它吧。




# Linkedlist

上一节介绍 ArrayList 的时候，我们先介绍了其继承类及实现的接口，类的成员变量，而后介绍了构造方法，常见的使用方法，最后判断其是否为线性安全，及其替代类。  

那么本节我们同样采取相同的思路，我们就先来看下它的继承是实现接口吧。

## Linkedlist 的继承与实现接口

Linkedlist 继承于 AbstractSequentialList ，而 AbstractSequentialList 又继承于 AbstractList ，这个类是不是很熟悉？ 是的，ArrayList 同样继承于 AbstractList ， 所以 LinkedList 会有大部分方法和 ArrayList 相似。并且 Linkedlist 实现了以下接口：


