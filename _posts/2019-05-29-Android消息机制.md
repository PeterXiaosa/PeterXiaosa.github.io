---
layout: post
title: Android消息机制浅析
categories: Android
description: 分析Android消息机制的主要类
keywords: Android、Handler、Looper、MessageQueue
---

Android的消息机制主要是Handler的运行原理，弄清楚了Handler是如何运行的也就大致清除了Android的消息机制是如何运行的。而Handler内部中又紧紧包含了Looper和MessageQueue，因此我们可以根据Handler源码依次顺藤摸瓜，弄清Looper和MessageQueue原理，从而明白消息机制。但我们需要先了解Handler，Looper，MessageQueue，Message是什么。


### Handler
为了了解Handler，我们先来看一下Handler的初始化函数做了哪些操作。
```java
    public Handler(Callback callback, boolean async) {
        ......
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```
可以看到Handler中存储了当前线程的Looper，同时可以看到如果该线程中如果没有Looper的话，创建Handler的时候便会抛出异常。Handler中除了Looper，还存储了Looper中的消息队列。

### Looper
在Handler的构造函数中我们看到了获取了Looper对象存入其中，那我们先来看下`myLooper`是怎么获取的。
```java
    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }
```
此处的sThreadLocal 是一个ThreadLocal对象，存储着当前线程的Looper。ThreadLocal是一个在线程之间存储数据的类。内部采用了Map映射的方法。<key(当前线程), Value(存储的数据)>


### MessageQueue
消息队列其实并不是采用数据结构中的队列来实现的，采用了一个单链表的形式来存储消息的。MessageQueue中有一个Message类型的局部变量mMessage，是一个单链表的形式来存储消息的。


### 消息传递流程
Handler 有两种方式可以向MessageQueue 发送消息，一种是post ，另外一种是sendMessage。但其实post 实际上也是通过sendMessage来发送消息的。我们可以来看下源码。

```java
    public final boolean post(Runnable r)
    {
       return  sendMessageDelayed(getPostMessage(r), 0);
    }
    
    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }
```
我们可以看到Runnable 通过`getPostMessage`来将Runnable封装成了一个Message 然后再发送出去。我们再来看下`sendMessage`。
```java
    public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }
```
`sendMessage`直接将message 发送出去，而post 我们通过源码查看，发现其本质也是和`sendMessage`一样发送Message。所以我们可以再来看一下Handler 如何发送消息。
```java
    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```
Handler 最终调用`sendMessageAtTime`来发送消息的，我们可以看到其实Handler只执行了`enqueueMessage`函数，我们再来看下这个函数。
```Java
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```
这个函数把Handler 存进了Message中的target中了，然后将message插入了消息队列中。我们紧接着到MessageQueue文件中看下`enqueueMessage`代码。
```Java
    boolean enqueueMessage(Message msg, long when) {
        ......
        synchronized (this) {
            ......
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```
在if else 中的判断中，if 中如果消息队列中没有消息了，那么将我们插入的消息作为消息头，并且需要去唤醒Looper。在else 中则是将msg插入到队列尾部。（因为本人算法不好，所以是用笔写写来理清的else中的逻辑）`enqueueMessage`执行完之后消息便插入MessageQueue的尾部了，所以接下来我们要看一下Looper中`loop`是如何从MessageQueue中取出消息来处理的。

```java
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;
        ......

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
            ......
            try {
                msg.target.dispatchMessage(msg);
                end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            ......
            msg.recycleUnchecked();
        }
    }
```
`loop`中从当前Looper 的消息队列中不断拿消息，然后将消息丢给`msg.target`的`diapatchMessage`来处理，而之前Handler 的`enqueueMessage`中有介绍过，Handler 把数据给消息队列之前将自己存入了Message 的 target 中，所以此处是把消息给发送它的Handler来处理，这样消息的处理就转移到了创建Handler的线程中来了。（而我们的Handler一般都是通过`getMainLooper`来创立的，即主线程的Handler，即将消息传入到主线程来处理了）  

接着，我们再来看下Handler 是如何处理消息的。
```java
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
    
    private static void handleCallback(Message message) {
        message.callback.run();
    }
```
首先看消息中的callback是不是空，而我们有说过Handler post Runnable的时候我们有通过`getPostMessage`把Runnable封装成消息，封装的时候就将Runnable存入了消息的callback中，所以此处消息中的callback就是我们之前post的Runnable对象，在`handlerCallback`中我们看到将Runnable运行在了主线程。 另外一种情况，如果我们是直接发送消息的那么我们就会调用`mCallback.handlerMessage`来处理消息，而`handleMessage`为我们创建Handler的时候重写的函数，这样也将消息在主线程中处理了。  

至此，我们已经从Handler发送消息到MessageQueue，然后Looper从MessageQueue中取消息进行处理的全部流程都梳理了一遍，这也就是Android消息机制的运行主线。文中可能有些讲解不当的地方，烦请大家指教。