---
layout: post
title: Java中关键字final和static比较
categories: Java
description: 比较final和static
keywords: final、static
---

本文主要介绍针对Java关键字final和static，在类，方法，变量三个方便来比较这两个关键字使用的区别。  


**目录**

* TOC
{:toc}

### final

#### final修饰的类
* final修饰的类无法被继承，并且final类中的方法默认为final类型。一般用于类的设计，当不想类被继承确保安全时，可以使用final 修饰类。

#### final修饰的方法
* final修饰的方法无法被重写(`override`)，当想要保持父类方法无法被子类重写时，可使用final修饰父类方法。


#### final修饰的变量
* final修饰的变量称为常量。存储在虚拟机的方法区中。final常量只能被赋值一次，一般显示在初始化时赋值，但也可以在之后赋值。final修饰引用时，引用恒定不变。但引用所对应的值确实可以改变的。也同样适用于数组。



### static

#### static修饰的类
* static静态类在类加载的时候便会被编译。用于静态内部类。只能访问静态方法和静态变量。
 
#### static修饰的方法
* static静态方法不依赖于类对象。不需要通过类对象.方法名来调用方法，只需要使用类名.方法名就可以调用类的静态方法。静态方法中也只能调用静态方法和静态变量。

#### static修饰的变量
* static静态变量存放在虚拟机的方法区中。单独分配一块内存地址用于存放static静态变量。在类初次被加载的时候会被初始化。

#### static代码块
* 静态代码块在类加载的时候便会运行。可以将一些只需要执行一次的初始化动作放在static代码块中。


### final和static总结
 type | static | final
-|-|-
Class |静态内部类|无法被继承
function|不依赖类对象|无法被重写|
variable|单独分配一块内存，类加载时初始化。|只能被赋值一次
