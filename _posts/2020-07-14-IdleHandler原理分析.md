---
layout: post
title: IdleHandler原理分析
categories: [Android, Handler]
description: some word here
keywords: Android, Handler， IdleHandler
---

## IdleHandler
之前的叙述中我们有介绍过[Android消息机制](https://peterxiaosa.github.io/2019/05/29/Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6/)，但当时还有一个小尾巴我们没有介绍到，当时我也没太注意，感觉不太重要，最近看 Matrix 源码时见到了 IdleHandler， 所以又重新回头拾起来给大家介绍一下。  

我想就三个方面来介绍 IdleHandler。首先，我们需要分析 IdleHandler 源码来知道它的工作原理。其次，我们要知道它有什么作用，使用它可以用来做什么，有哪些使用场景。最后，我们再来看下系统源码中哪里使用到了 IdleHandler，以及用它是用来做什么的，从而巩固第二点对于 IdleHandler 作用的分析。  

那么首先我们就先来通过源码分析了解一下 IdleHandler 的工作原理吧。

### IdleHandler的源码分析
首先，我们来看下 IdleHandler 的添加和移除。

``` java
    public void addIdleHandler(@NonNull IdleHandler handler) {
        if (handler == null) {
            throw new NullPointerException("Can't add a null IdleHandler");
        }
        synchronized (this) {
            mIdleHandlers.add(handler);
        }
    }

    public void removeIdleHandler(@NonNull IdleHandler handler) {
        synchronized (this) {
            mIdleHandlers.remove(handler);
        }
    }
```
我们可以通过当前线程的 Looper 获取消息队列，然后调用 `addIdleHandler(IdleHandler)` 和 `removeIdleHandler(IdleHandler)` 添加和移除 IdleHandler。  

知道了如何添加和移除 IdleHandler，我们再来看下 IdleHandler 的使用。

在 Looper 的 `loop()`方法中，会调用 MessageQueue 中的 `next()` 来取消息处理，而 IdleHandler 就是在此方法中使用了。

``` java
    Message next() {
        // ......

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // ......

                // 1
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                // 2
                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                // 3
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null;

                boolean keep = false;
                try {
                    // 4
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                // 5
                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```
`next()`中取消息的部分我们省略了，具体逻辑可参考之前介绍的[Android消息机制](https://peterxiaosa.github.io/2019/05/29/Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6/)。现在我们只分析有关 IdleHandler 的这一段代码。  

首先 `pendingIdleHandlerCount` 默认为-1，所以注释1中的 `pendingIdleHandlerCount<0` 条件是成立的，可是还有一个 `mMessages == null || now < mMessages.when` 条件，这个条件的意思是当前消息队列没有消息了，或者说消息队列有消息，但是消息不是现在处理的(之后某个时间再处理)。通俗说就是当前没有可处理的消息的时候，就会进入注释1中计算 IdleHandler 的数量。也就是说，IdleHandler是用在消息队列闲暇的时候的，当消息队列中有消息的时候 IdlerHandler 不会起作用，只有在消息队列处理完消息的时候才会发生作用。

接着在注释2中，创建了一个数量至少为4的 IdleHandler 数组。并且将消息队列中的 IdleHandler 全部赋值为数组中的元素。  

注释3中，获取 IdleHandler，之后释放数组中的 IdleHandler 引用。

注释4中，调用 IdleHandler的 `queueIdle()` 函数，并且返回一个 bool 值。  

注释5中，利用 `queueIdle()`的返回值来判断是否需要移除 IdleHandler, 当 `queueIdle()` 返回 false 的时候，消息队列会移除该 IdleHandler, 返回为 true时，则继续保留。  

从上述的代码中我们可以了解到 IdleHandler 的几点特性。

* IdleHandler 是在消息队列当前无可用消息的时候发生作用的，如果你想在消息队列空暇时做一些处理，那么你就可以在当前线程的消息队列中添加 IdleHandler，并重写它的 `queueIdle()` 函数。
* IdleHandler 作用次数可为一次或者多次。这取决于它的 `queueIdle()` 的返回值，如果 `queueIdle()` 返回 false, 那么消息队列第一次空暇调用完 `queueIdle()` 之后便会将该 IdleHandler 移除。如果返回 true， 那意味着每次只要消息队列空闲，就会调用一次 `queueIdle()`。

可能有些小伙伴这个时候会有些疑问，如果 `queueIdle()` 返回 true 的时候，如果消息队列一直没有消息处于空闲状态，是不是就会无限循环调用 `queueIdle()` ？ 回答这个问题之前，我们需要回想起以前介绍的在 `next()`中的函数 `nativePollOnce()`。我们之前介绍过，这个函数在 native 层会将线程阻塞，等有新事件来临的时候再进行唤醒，所以就不会出现我们之前猜测的无限调用 `queueIdle()` 的问题。

### IdleHandler 有什么作用呢？

那从源码角度讨论完 IdleHandler ，了解了它的特性和工作原理，接下来我们就来分析一下它有什么作用。  

其实分析 IdleHandler 的作用也是需要从它的特性和工作原理来思考的。 
首先，IdleHandler 是在消息队列当前无可用消息的时候发生作用的，那么我们就可以猜测 IdleHandler 是不是可用来执行一些优化动作。比如，处理一些不是十分重要的初始化操作。在启动 Activity 的时候，如果将一些不太重要的初始化动作 `onCreate()`、`onResume()`等函数中可能会造成页面显示缓慢的问题，影响应用启动速度。而如果将这些操作使用 IdleHandler 的 `queueIdle()` 来进行初始化，那么就可以在线程空闲的时候来进行这些初始化动作，加快用户看到界面的速度，从而提高用户体验。但同时需要注意一点， IdleHandler 依赖于消息队列中的消息，如果当前一直有消息进入消息队列，那么 IdleHandler 可能就一直都无法有执行的机会。  

其次，我们有时候想获取某个控件的宽高，又或者是想要某个View绘制完之后添加依赖于这个View的View，那么就可以使用 IdleHandler 来进行获取宽高或者添加View。 当然也可以使用 `View.post()`来实现，区别是前者是在消息队列空闲的时候执行， 后者是作为一个消息来执行。  

LeakCanary 中也有使用到 IdleHandler， LeakCanary 2.0中的源码使用Kotlin编写的，还没有研究。等之后看了再补上。

### IdleHandler 在系统源码中的运用
 IdleHandler 在系统源码中也有过使用，其中 `GcIdler`便实现了 `IdleHandler` 接口。我们来看下 `GcIdle` 是如何工作的。

 ``` java
    final class GcIdler implements MessageQueue.IdleHandler {
        @Override
        public final boolean queueIdle() {
            doGcIfNeeded();
            return false;
        }
    }

    void doGcIfNeeded() {
        mGcIdlerScheduled = false;
        final long now = SystemClock.uptimeMillis();
        //Slog.i(TAG, "**** WE MIGHT WANT TO GC: then=" + Binder.getLastGcTime()
        //        + "m now=" + now);
        if ((BinderInternal.getLastGcTime()+MIN_TIME_BETWEEN_GCS) < now) {
            //Slog.i(TAG, "**** WE DO, WE DO WANT TO GC!");
            BinderInternal.forceGc("bg");
        }
    }
```

`GcIdler` 用来实现强制性GC操作的。当现在时间超过了上一次GC的 `MIN_TIME_BETWEEN_GCS == 5000`， 也即离上一次GC超过5秒之后就会再一次强制性触发GC, 并且每个GcIdler 返回 false，只触发一次。而 `GcIdler`是怎么添加的呢？我们可以看下。

``` java
    void scheduleGcIdler() {
        if (!mGcIdlerScheduled) {
            mGcIdlerScheduled = true;
            Looper.myQueue().addIdleHandler(mGcIdler);
        }
        mH.removeMessages(H.GC_WHEN_IDLE);
    }
```

通过调用 `scheduleGcIdler()` 来进行触发。而 `scheduleGcIdler()`又是通过 ActivityThread 中的 Handler `H`发送消息触发的。再往上追溯，我们可以知道是在 AMS 中的这两个方法调用之后触发：
* doLowMemReportIfNeededLocked
* activityIdle  

所以Android 会在内存不够的时候使用 IdleHandler 来进行强制性GC 优化。或者可能当 ActivityThread 的 `handleResumeActivity`方法被调用时触发的。


### 引用

* [关于IdleHandler的疑惑解答，帮你干掉一个面试题](https://zhuanlan.zhihu.com/p/78804097)
* [Android IdleHandler 原理浅析](https://www.cnblogs.com/mingfeng002/p/12091628.html)
* [面试官：“看你简历上写熟悉 Handler 机制，那聊聊 IdleHandler 吧？”](https://juejin.im/post/5e4de2f2f265da572d12b78f)