---
layout: post
title: HashMap的源码分析
categories: Java
description: HashMap是我们经常用的数据结构，采用了key-value的方式来存储数据。在JDK 1.8的版本中也是对其做了优化修改，现在我们来通过在JDK 1.8的环境下的源代码分析一下HashMap的工作原理。及相比于1.7，在1.8中的优化。
keywords: Java，HashMap，ReSize，hash
---

HashMap是我们经常用的数据结构，采用了key-value的方式来存储数据。在JDK 1.8的版本中也是对其做了优化修改，现在我们来通过在JDK 1.8的环境下的源代码分析一下HashMap的工作原理。及相比于1.7，在1.8中的优化。