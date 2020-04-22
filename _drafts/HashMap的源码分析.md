---
layout: post
title: HashMap的源码分析
categories: Java
description: HashMap是我们经常用的数据结构，采用了key-value的方式来存储数据。在JDK 1.8的版本中也是对其做了优化修改，现在我们来通过在JDK 1.8的环境下的源代码分析一下HashMap的工作原理。及相比于1.7，在1.8中的优化。
keywords: Java，HashMap，ReSize，hash，红黑树
---

HashMap是我们经常用的数据结构，采用了key-value的方式来存储数据。在使用设计的过程中牵涉了许多的数据结构及知识点，数组+链表，扩容，头插尾插，hash函数等。JDK 1.8的版本中也是对其做了优化修改，现在我们来通过在JDK 1.8的环境下的源代码分析一下HashMap的工作原理，及相比于1.7，在1.8中的优化。

**目录**

* TOC
{:toc}

# HashMap的源码分析

## 概述
HashMap是我们经常用的数据结构，采用了key-value的方式来存储数据。在使用设计的过程中牵涉了许多的数据结构及知识点，数组+链表，扩容，头插尾插，hash函数等。JDK 1.8的版本中也是对其做了优化修改，现在我们来通过在JDK 1.8的环境下的源代码分析一下HashMap的工作原理，及相比于1.7，在1.8中的优化。本文基于JDK 1.8中源代码，比较与JDK 1.7中代码的时候会额外说明。同时，本文的描述从构造函数及节点等基本数据类型开始，然后再通过我们常用的`put()`、`remove()`等常用函数来逐步介绍了解，碰到的一步步状况作者在设计时都做了什么，为什么是这样设计的，而不是直接将一堆结论罗列出来讲解。


### Hashmap结构
首先，我们来看下初始化HashMap的时候的构造函数`new HashMap()`。

``` java
    /**
     * The load factor used when none specified in constructor.
     */
    // 1
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```
Hashmap的初始化中，在注释1处设置了Hashmap的默认负载因子`DEFAULT_LOAD_FACTOR`为 0.75f。接下来，我们再来看下Hashmap中的一个基本结构-Hashmap节点。

### Hashmap节点
HashMap中有一个静态内部类`Node`，为Hashmap的节点。我们来看下`Node`中的结构。
``` java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
        //......
    }
```
从该静态类中可以看到，节点有4个属性值。第一个为key值的hash值hash，由key值经过hash函数得来的。第二个为相对应的key值。第三个为Value值。最后一个值为下一个节点Next值，所以可以看出`Node`其实是一个链表的结构。

### HashMap常用方法
介绍完Hashmap的初始化后，我们来看下Hashmap的常用函数的用法。

#### put(K key, V value)
我们直接来看下`put`方法的源代码。

``` java 
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            // 1.1
            n = (tab = resize()).length;
            // 1.2
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 1.3
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 1.4
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 1.5
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // 1.6
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            // 1.7
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
这个函数信息量比较大，做的事情比较多。我们先来看下注释1.1。默认table是为一个空数组，那么函数会首次调用`put()`函数的时候会直接调用`resize()`函数，这个函数称为hashmap的扩容函数。从这个函数中我们可以引出hashmap的扩容机制。

### Hashmap的哈希及索引算法
Hashmap通过散列表函数`hash(key)`计算`key`的哈希值。通过hash值来计算`Node`节点所在数组的索引。来看下是如何计算hash值及如何计算索引的。

``` java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

    // 1
    // n = tab.length
    // i = (n - 1) & hash
```

计算hash值过程很简单，只有一行代码。取key的`hashCode`，将`hashCode`的低16位和高16位进行异或运算。而key的`hashCode()`方法是由自己重写的。`String`类型的hashCode在Java中有默认重写方法。所以从这里我们可以知道，作为Hashmap的key值则必须重写`hashCode()`方法。  
同时，`hashCode`为int类型，4个字节有32位，取低16位与高16位进行异或运算是为了尽可能的让`hashCode`值的高位和低位参与运算。因为注释1中将数组长度减1和`hash`值做与运算来确定`Node`节点所在数组的索引。高低16位进行异或这种做法可以在数组长度`n`值较小时也能让尽可能多的位值参与运算。同时取位运算而不通过`mod`模运算来计算余数是为了更高效。  
除此之外，将`hash`值与`n-1`取与运算，是因为数组的长度都是2的幂，所以`n-1`都为1。类似于`01111...`这种形式，1的个数为2的幂数。所以设计hashmap数组的时候故意将数组长度设置为2的幂是为了散列值计算出来的索引更均匀的分布在数组中，减少hash碰撞。

### Hashmap的扩容机制
由上述`put()`方法，我们接触到扩容。当hashmap容量不足以储存新数据时，便要采用扩容来扩大Hashmap容量。重点我们来看下上述提到的`resize()`函数。

``` java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 2.1
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            // 2.2
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            // 2.3
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            // 2.4
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```
初始化数组的时候使用扩容会进入注释2.2处，数组容量采用默认容量`DEFAULT_INITIAL_CAPACITY`16,`threshold`为默认负载因子乘以容量。`threshold`的作用为判断hashmap是否需要扩容，当hashmap中的Node节点数`size`大于`threshold`时，那么hashmap就需要扩容了，这一点在`put`中的源码有体现，我们可以之后来验证它。

而当hashmap节点`size`到一定容量时也会触发扩容，此时就会进入注释2.1处。数组长度通过位运算进行乘2扩容，同时由于负载因子未变，`threshold = capacity * loadfactor`也会扩大一倍。所以也通过位运算左移一位完成乘2操作。

完成了`capacity`和`threshold`的更新之后，注释2.3处便新建一个新容量的数组，在注释2.4处进行节点元素的迁移。从将旧数组中的元素迁移到新数组中。  

![hashmap_resize_old](/images/posts/java/hashmap_resize_old.jpg)

