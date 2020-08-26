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

* List : 实现了 List 接口，表明它也是一个列表，是 Java Collection Framework 中的一员。

* Deque ：这个接口又继承于 Queue ，具有队列的方法，是一个支持元素在双端插入和删除的线性集合。同时记住它的发音为 "deck", 不要闹笑话哦。

* Cloneable ：这个接口表明它同样具有拷贝的能力，可以进行深浅拷贝。

*  Serializable : 这个接口表明它具有序列化，可以进行持续化存储或网络传输。

## Linkedlist 的类变量

``` java
    
    // 内部节点的数量
    transient int size = 0;
    // 头节点
    transient Node<E> first;
    // 尾节点
    transient Node<E> last;

    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

Linkedlist 的类变量很少，只有3个，但其中还有一个静态内部类 Node，我们依次来看下。  

* size : 表明内部节点的个数，或者说插入元素的个数，一个节点代表一个元素。

* first : 头结点，这个变量指向头节点，因为 Linkedlist 是一个双向链表，至于为什么是双向链表，接下来我们可以从 Node 节点中知道。

* last : 尾节点，这个变量指向链表的尾部节点。

* Node : 静态内部类，从它的变量 next 和 prev 我们就可以知道，这是一个双向链表类，其中 next 指向该节点的下一个节点，prev 指向该节点的上一个节点。而 item 即为该节点存储的元素。

## Linkedlist 的使用方法

Linkedlist 是一个双向链表，所以大部分方法都是在操作双向链表的。同样，我们通过最基本的增删改查操作来介绍 Linkedlist 的使用方法吧。

### 头插法

由于 Linkedlist 双向链表的特性，所以可以从头插入元素，Linkedlist 从头插入元素的方法有几个，分别为 `push()`, `addFirst()`, `offerFirst()`。我们可以先通过图来看下它是如何插入的。  

![Linkedlist的头插法](http://xiaosalovejie.top/images/head_insert.png)  

从图中我们可以看到，左边是链表头 head 节点，右边是链表尾 tail 节点。接下来，我们就看下头插的源码是如何实现的吧。实现头插法的内部其实都实现了 `linkFirst()`。

``` java
    /**
     * Links e as first element.
     */
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
```
头插时，将要插入的元素 e 封装成 Node 节点。然后将 Node 节点和之前的头结点 first 连接起来，并且将新节点复制尾 first。

### 尾插法

双向链表除了头插，还可以尾插。实现的尾插方法有 `offer()`, `offerLast()`, `addLast()`, `add()`。我们也可以使用一张图来表示如何尾插的。

![Linkedlist的尾插法](http://xiaosalovejie.top/images/end_insert.png)  

尾插的方法内部也都是 `linkLast()`, 再来看下源码是如何实现的。

``` java
    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

尾插将要插入的元素封装成节点 newNode, 然后将 newNode 连接到尾节点的 last 之后，然后将尾节点 last 指向节点 newNode。

接下来还有头除法，和尾除法。同样，我们使用图来更好的理解。

### 头除法

相关的方法有`pop()`,`removeFirst()`,`remove()`, `poll()`,`pollFirst()`。

![Linkedlist的头除法](http://xiaosalovejie.top/images/head_remove.png)  

### 尾除法
相关的方法有`removeLast()`, `pollLast()`。

![Linkedlist的头除法](http://xiaosalovejie.top/images/end_remove.png)  

从头移除节点与从尾移除节点的源码实现和头插尾插的原理差不多，这里就不再展示了。头除是将 first 节点移除，然后将 first 节点指向原 first 节点后的那个节点。尾除原理亦然。  

接下来，我们看下查的方法。

### get(int index)

``` java
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }

    /**
     * Returns the (non-null) Node at the specified element index.
     */
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```
`get()`方法通过索引 index 来取元素，内部直接调用 `node()`方法。方法内部通过 `size >> 1`即元素的一半来判断，如果在前半段那么就从头一个个节点遍历过去找到第 index 个元素，如果在后半段从就尾一个个遍历，因为是一个双向链表，所以可以从头或从尾遍历。  

`set(int index, E element)`也是通过同样的道理来更新元素值的，通过`index`来调用`node()`方法找到节点，然后用新元素值更新到节点中的值。  

## Linkedlist 的线程安全

我们在上述源码中可是，不管是增还是删都没有看到相关的锁。所以说 `LinkedList`是非线程安全的。而如果我们想在多并发的场景下使用这样的双向链表应该怎么办呢？ 我们可以采用 `Collections.synchronizedList()`，`Collections.synchronizedCollection()`或者使用 `LinkedBlockingQueue`来进行多并发场景下的双向链表的使用。

## 分析

* Linkedlist 内部是一个双向链表结构，所以我们可以用它来实现栈或者队列的结构，使用栈时，我们可以从头插头除使用`push()`和`pop()`方法来分别表示入栈和出栈的过程。而使用队列时我们可以用尾插和头除，即`offer()`和`poll()`方法来分别表示入队和出队的过程。 

* 结构决定了性质。由上述的增删源码中我们可以知道，当新增的时候，只需要重新构建一个新节点然后分别将新节点接入到原链表中,移除则是直接在原链表的头部或者尾部直接将节点去除，然后使用 head 或者 tail 节点重新指向前一个或者后一个节点即可，十分迅速。而对比 ArrayList 则需要扩容，元素在数组中迁移等，所以 Linkedlist 更适合插入删除操作。而查询更新的话，LinkedList 则需要一个个节点遍历过去插，所以效率无法和 ArrayList 比较。

## 总结

* 1.Linkedlist为非线程安全类，想使用线程安全的话可以借助 `Collections`工具类或者 `LinkedBlockingQueue`类。

* 2.Linkedlist是双向链表结构。所以对元素的插入删除很快，对元素的查询修改效率慢。适合于修改多查询少的场景。

* 3.Linkedlist 可在链表双端进行增删操作，所以可以用来实现队和栈的功能。
