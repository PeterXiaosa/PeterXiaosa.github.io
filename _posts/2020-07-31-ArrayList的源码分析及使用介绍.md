---
layout: post
title: ArrayList的源码分析及使用介绍
categories: [Java,ArrayList]
description: 对ArrayList的源码分析和使用介绍，从继承，接口，使用方法，fail-fast，线程安全方面介绍理解。
keywords: ArrayList,List,Java,fail-fast
---

今天我们来对最常用的Java集合中的ArrayList做一次源码的分析介绍，从源码角度看ArrayList的使用和方法。ArrayList 是最普通最普通的一种Java集合，所以说ArrayList底层源码的准备还是有必要的，否则和小伙伴沟通的时候只会用不会说，那岂不是很尴尬= =

# ArrayList

对于ArrayList的分析，我们先从它的继承类，实现的接口开始接触，然后再看看ArrayList的构造方法，其中的类变量，了解类变量。接着再来看下它的常用方法都是如何实现的，其次分析一下和它其中的 modCount 变量相关的 fail-fast 机制。最后，讨论分析 ArrayList 是否线程安全，如果不安全是否有其替代类。  

下面我们就来逐个分析介绍。

## ArrayList 的继承与接口  

ArrayList 继承于抽象类 AbstractList，为 Java Collections Framework 中的一员。分别实现了 List, RandomAcess, Cloneable, Serializable接口。  

* List 接口表明，ArrayList 具有 List 操作的特性。  

* RandomAcess 接口表明， ArrayList 具有随机访问的特性，这个接口没有任何方法实现，为声明接口。实现了该接口的类表明了具有随机访问的能力，可使用 for循环更快的遍历，而不是使用迭代器循环。在 Java 的 List 集合中的不同子类， 根据是否实现 RandomAccess 接口采用不同的遍历方式，能够提高性能。  

* Cloneable 接口表明， ArrayList 可进行深浅拷贝(克隆)。  

* Serializable 接口表明， ArrayList 可进行序列化操作，可将 ArrayList 转化为二进制字节流进行网络传输或者持久化操作。  

上述即 ArrayList 的继承和实现的接口，不同的实现接口分别表明了它具有不同的功能，接下来我们就来先了解一下 ArrayList 的各种变量吧。

## ArrayList 的类变量

我们直接从 ArrayList 的源码中来看它的类变量。

``` java
    //  序列化ID
    private static final long serialVersionUID = 8683452581122892189L;

    // 默认初始化数组的大小，
    private static final int DEFAULT_CAPACITY = 10;

    // 空的数组，当初始化表明 capacity 为0时，arrayList 即使用此数组
    private static final Object[] EMPTY_ELEMENTDATA = {};


    // 空的数组，当未表明 capacity 时，arrayList 使用此数组，并且大小使用 DEFAULT_CAPACITY
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};


    // arrayList 中的动态可变长数组
    transient Object[] elementData; // non-private to simplify nested class access


    // arrayList 中数组元素的个数
    private int size;
```

ArrayList 中的类变量就上述中的几个，现在我们来分别介绍：

* serialVersionUID : 序列化ID，存在该ID时，ArrayList 即可正常使用序列化和反序列化，当二进制字节流改变时会表明无法恢复到正确的数据。  

* DEFAULT_CAPACITY : 默认初始化数组大小，当 new 一个 ArrayList 对象时，如果没有指明 ArrayList 数组大小，则默认采用 DEFAULT_CAPACITY， 即为10。  

* EMPTY_ELEMENTDATA : 空元素数组，当构造 ArrayList 时，如果初始化大小表明为0，那么 ArrayList 初始化数组就会使用空数组，当下一次 ArrayList 进行 add 操作的时候，数组进行扩容时，数组长度即会变为1.  

* DEFAULTCAPACITY_EMPTY_ELEMENTDATA : 默认初始化空元素数组，和 EMPTY_ELEMENTDATA 一样都是空数组，区别是 EMPTY_ELEMENTDATA 是构造时指定了初始化大小0， 而 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 则是没有指定初始化大小，例如 `new ArrayList()<>` ， 当数组为 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 时，下一次 add 操作时，数组进行扩容会自动将长度直接扩大为 DEFAULT_CAPACITY。  

* elementData ： 动态可变长数组，ArrayList 中用于存储元素数据的数组，当数组长度不够继续添加新元素时会自动进行扩容，每一次扩容会将数组长度扩大为原来的1.5倍。

* size : ArrayList 内部数组存储的元素个数，这里需要注意的是这里 size 是指元素个数，而不是数组长度。当我们进行 set, remvoe 操作时，是通过 index 和 size 进行比较判断是否越界，而不是和内部数组长度来比较的。

ArrayList 的类变量就上述这些，接下来我们来分析下 ArrayList 中构造方法，从源码角度去解析。


## ArrayList 的构造方法
ArrayList的构造方法有三个，一个是无参，一个是初始化大小，一个是`Collection`集合。我们分别来看下这三个构造函数。  

### 无参的构造函数
``` java
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```
可以看到，当无参构造时，ArrayList的内部数组会赋值为 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 。当使用 `add()` 方法第一次添加元素时，即会初始化数组大小为 DEFAULT_CAPACITY 。

### 带初始化大小的构造函数
``` java
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```
当传入的初始化数组大小大于0时，则新建一个大小为 `initialCapacity` 大小的 Object 数组。当传入的初始化数组大小未0时，则赋值为 EMPTY_ELEMENTDATA 空数组。那小于0的时候就直接抛异常了，因为不允许大小为负数。

### 带 Collection 集合的构造函数
``` java
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
使用集合传入来构建 ArrayList 时将集合转化为数组，然后依次将集合中的元素拷贝至数组中。

这三个构造函数都比较简单，主要是还是来看下 ArrayList 中常用的方法。

## ArrayList 的常用方法

对 ArrayList 的常用方法莫过于增删改查，分别为 add(), remove(), set(), get()。同时除了这几个外，常用的还有`size()`,`isEmpty()`等。

### add(E e)
首先我们直接看最最最常用的`add(E e)`方法。

``` java
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 1
        elementData[size++] = e;
        return true;
    }

    private void ensureCapacityInternal(int minCapacity) {
        // 2
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }    

    private void ensureExplicitCapacity(int minCapacity) {
        // 3
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            // 4
            grow(minCapacity);
    }
```
`add()`方法中调用了一个`ensureCapacityInternal()`方法，然后接着在注释1中就将要添加的元素`e`赋值给数组中，并更新元素个数 `size` 的值，最后返回一个 `true`表明添加成功。  

在 `ensureCapacityInternal()`方法的注释2中可以看到，如果 ArrayList 的内部数组为 DEFAULTCAPACITY_EMPTY_ELEMENTDATA， 那么就将数组大小赋值为默认的长度大小 `DEFAULT_CAPACITY`，也即之前讲述 ArrayList 类变量中的值。然后接下来执行了重要的 `ensureExplicitCapacity()` 方法。  

`ensureExplicitCapacity()` 方法中会先将变量 `modCount` 加1，这个 `modCount`有什么用呢？ `modCount`是用在 `Iterator`的迭代中的，利用 `modCount` 和 `expectedModCount` 比较，如果不一致(例如多线程操作时)则抛出 `ConcurrentModificationException` 异常。而 `modCount` 会在每次对 ArrayList 数据元素有改动时，及 add, remove, addAll, removeAll, clear 等，将 `modCount`加1。 不能在顺序遍历 ArrayList 的时候去修改 ArrayList。否则直接抛异常，这里其实是一种 fail-fast 机制，是一种 Java集合的错误机制。  

fail-fast 是一种错误检测机制，只能用来检错不能纠错，因为 JDK 并不能保证 fail-fast机制一定会发生，多线程情况下它只是尽最大努力将错误抛出。如果想在多并发情况下使用，那么可以使用 java.util.concurrent 下的集合类，或者 Vector。

回到 `ensureExplicitCapacity()` 方法，在修改完 `modCount` 之后如果需要的数组长度超过目前 ArrayList 中的内部数组长度，那么就进行注释4中的 `grow()` 扩容操作。

``` java
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 1
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        // 2
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
注释1中就是扩容后的新值，为原来值的1.5倍，这里采用了位运算，效率比÷2 高。在计算完扩容后的新值后在注释2中，创建新数组并且将原数组中的值分别拷贝过去。  

所以在`add()`方法中，先更新 `modCount` 变量，然后对数组进行扩容(如果需要的话)，然后将新插入的元素赋值在新数组中，最后 `return true` 说明元素插入成功。

### Remove(int index)
``` java
    public E remove(int index) {
        // 1
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        // 2
        modCount++;
        E oldValue = (E) elementData[index];

        int numMoved = size - index - 1;
        if (numMoved > 0)
            // 3
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```
`remove()`中的注释1中先将需要移除位置的索引 `index` 和数组元素个数 `size`比较，需要注意，这里是和数组元素个数比较不是和数组长度比较。 然后再注释2中先获取到该位置下的旧元素值，注释3中将从 `index+1` 处的元素到末尾的元素依次赋值为从 `index` 处到 `末尾-1` 处。就依次把后面的元素往前挪一位，这样就覆盖了之前的元素，从而达到了移除 `index` 处元素的效果。最后将 `size` 更新，清除最后多的一位元素，方便GC回收。完成了移除数据操作之后返回该处的数组原值。

### Remove(Object o)
``` java
   public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```
这个函数和上一个`remove(int index)` 其实差不多，但是还是需要单独说一下。这里是依次顺序遍历数组，也即是移除顺序第一个为 o 的元素，找到了为 o 的元素就进行移除，并且将 size 更新。  

### set(int index, E element)
``` java
    public E set(int index, E element) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        E oldValue = (E) elementData[index];
        elementData[index] = element;
        return oldValue;
    }
```
`set()`方法就更简单了，首先判断需要更新的值是否越界，越界直接抛异常。否则更新数组中的值，返回原值。  


### get(int index)
``` java
    public E get(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        return (E) elementData[index];
    }
```
这个方法比 `set()` 还简单，直接获取数组中该位置的值返回出去。

## 分析

* 从上面常用的方法中，我们其实可以思考到一些问题。例如 `modCount`， 会和 `expectedModCount`比较，如果不一致则抛异常。所以说明 ArrayList 不能在多线程场景下使用，即为线程不安全。  

* 同时，我们发现每次使用 `add()` 的时候可能需要扩容，如果扩容的话就需要数据的迁移。使用 `remove()` 的时候，每次都需要使用后一个数据逐个覆盖前数据。在数据量小的时候可能区别不大，但如果在数据量较大的时候，每一次扩容操作以及移除操作的消耗就很大了。 所以 ArrayList 不适合在数据量大的时候使用。  

* 在发现对 ArrayList 的增删消耗很大后，我们可能会有些失望，那 ArrayList 有什么优势呢？否则我们为什么要用它。但其实有没有发现，我们去进行 `set()` 和 `get()` 的时候会很快，为什么很快呢？因为我们是对数组直接操作的，数组中有了索引之后可以直接定位该位置的元素。最根本的是因为，数组在内存分配中内存地址是连续的，所以查的时候时间复杂度为O(1)。 所以 ArrayList 适合改少查多的情景。

## 总结

* 1.ArrayList 内部是动态可变数组，数组长度根据添加的元素来决定，当内部数组元素到达一定数量时就会进行扩容，扩容后的新数组长度为原来长度的1.5倍。

* 2.ArrayList 为非线程安全集合类。如果想在多并发情景下使用，可以选择使用 Vector 或者 java.util.concurrent 下的集合类。

* 3.ArrayList 的增删操作在数据量大的时候消耗非常大，但改查操作很快。所以 ArrayList 适用于查多加少的情况。 大部分操作都是查的话，使用 ArrayList 效率会非常高。


## 引用

* [《我们一起进大厂》系列-ArrayList](https://juejin.im/post/6844904040359264270#heading-1)

* [ArrayList源码解析，老哥，来一起复习一哈？](https://mp.weixin.qq.com/s/3PNWmtS-bEZgZjd9wyMiDA)

