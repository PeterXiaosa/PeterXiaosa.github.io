---
layout: post
title: 起舞者的艺术-Choreographer
categories: [Android, UI渲染, Choreographer]
description: Android中Choreographer的工作原理。
keywords: Android,VSync,Choreographer
---

之前看Matrix源码，看到了在Matrix中的一个`UIThreadMonitor`中涉及到了对 IdleHandler以及 Choreographer 中的反射，没有看太懂。所以后又返回了去看 [IdleHandler](https://peterxiaosa.github.io/2020/07/14/IdleHandler%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90/)，Android闲时处理机制。了解了之后接着又来看与本文有关的`Choreographer`机制，所以本文就 Choreographer 源码做一些简单介绍和分析。

## Choreographer
Choreographer,中文翻译为编舞者。在Android的功能为接收VSYNC信号进行输入，动画，界面的刷新任务的实现。Android界面在 Choreographer 的引导下能有条理的显示出来，宛如舞者在屏幕上翩翩起舞一般。

接下来，我们还是向往常一样，先从源码的角度分析`Choreographer`的工作原理，然后通过它的原理知道它的作用以及我们在哪些场景下可以使用到`Choreographer`。

### Choreographer的获取
其实，我们可以发现`Choreographer`是一个单例，通过`getInstance()`来获取`Choreographer`的。而单例中并不是通过内部类或者说DLC来进行构造的，而是通过`ThreadLocal`来进行初始化获取的，我们可以来看下。

``` java
    public static Choreographer getInstance() {
        return sThreadInstance.get();
    }

    // Thread local storage for the choreographer.
    private static final ThreadLocal<Choreographer> sThreadInstance =
            new ThreadLocal<Choreographer>() {
        @Override
        protected Choreographer initialValue() {
            Looper looper = Looper.myLooper();
            if (looper == null) {
                throw new IllegalStateException("The current thread must have a looper!");
            }
            return new Choreographer(looper, VSYNC_SOURCE_APP);
        }
    };
```
由于使用`ThreadLocal`来进行存储的，就说明`Choreographer`的作用域为线程。各个线程拥有自己独立的`Choreographer`单例。从`initialValue()`函数来看，更确切的说应该是，各个具有`Looper`的线程都拥有自己独立的`Choreographer`单例。每个`Choreographer`单例的构造都需要`Looper`，我们来看下`Choreographer`和`Looper`又有什么联系呢？

### 构造函数

``` java
    private Choreographer(Looper looper, int vsyncSource) {
        mLooper = looper;
        mHandler = new FrameHandler(looper);
        mDisplayEventReceiver = USE_VSYNC
                ? new FrameDisplayEventReceiver(looper, vsyncSource)
                : null;
        mLastFrameTimeNanos = Long.MIN_VALUE;

        mFrameIntervalNanos = (long)(1000000000 / getRefreshRate());

        mCallbackQueues = new CallbackQueue[CALLBACK_LAST + 1];
        for (int i = 0; i <= CALLBACK_LAST; i++) {
            mCallbackQueues[i] = new CallbackQueue();
        }
    }
```
在构造函数中，保存了全局变量`mLooper`,并且通过`Looper`构造了`FrameHandler`和`FrameDisplayEventReceiver`。同时生成了一个包含不同任务种类的任务数组`mCallbackQueue`。这三个变量对`Choreographer`影响很大，我们先在这里介绍这三个变量作用，然后再进行介绍`Choreographer`的使用。

### FrameHandler

先来看下`FrameHandler`的构造。

``` java
    private final class FrameHandler extends Handler {
        public FrameHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_DO_FRAME:
                    doFrame(System.nanoTime(), 0);
                    break;
                case MSG_DO_SCHEDULE_VSYNC:
                    doScheduleVsync();
                    break;
                case MSG_DO_SCHEDULE_CALLBACK:
                    doScheduleCallback(msg.arg1);
                    break;
            }
        }
    }
```
由`Choreographer`的构造函数可知，`FrameHandler`的 looper 是`Choreographer`所在线程的 Looper. 也就是说，通过`FrameHandler`可将方法运行在`Choreographer`所在的线程。`FrameHandler`有三钟消息，分别为`MSG_DO_FRAME`、`MSG_DO_SCHEDULE_VSYNC`、`MSG_DO_SCHEDULE_CALLBACK`。 FrameHandler 主要是用来将任务执行转移到 `Choreographer` 所属线程来执行的。

### CallbackRecord

``` java
    private static final class CallbackRecord {
        // 1
        public CallbackRecord next;
        public long dueTime;
        public Object action; // Runnable or FrameCallback
        public Object token;

        public void run(long frameTimeNanos) {
            if (token == FRAME_CALLBACK_TOKEN) {
                ((FrameCallback)action).doFrame(frameTimeNanos);
            } else {
                ((Runnable)action).run();
            }
        }
    }
```
由注释1中的变量`next`可以知道，这是一个链表。类似于`Message`。其中`action`变量一般为`Runnable`为要执行的操作。所以一般一个`CallbackRecord`变量为一个任务链表，存储着某一类型的所有任务，根据`dueTime`时间来确定任务执行的先后顺序，这个和`Message`很相似。`Choreographer`中维护了一个`CallbackQueue`数组，`CallbackQueue`中维持着链表 CallbackRecord 头，数组中每个元素 CallbackQueue 都代表一种类型的任务。

### FrameDisplayEventReceiver

``` java
    private final class FrameDisplayEventReceiver extends DisplayEventReceiver
            implements Runnable {
        private boolean mHavePendingVsync;
        private long mTimestampNanos;
        private int mFrame;

        public FrameDisplayEventReceiver(Looper looper, int vsyncSource) {
            // 1
            super(looper, vsyncSource);
        }

        @Override
        public void onVsync(long timestampNanos, int builtInDisplayId, int frame) {
            // ......

            mTimestampNanos = timestampNanos;
            mFrame = frame;
            // 2
            Message msg = Message.obtain(mHandler, this);
            msg.setAsynchronous(true);
            mHandler.sendMessageAtTime(msg, timestampNanos / TimeUtils.NANOS_PER_MS);
        }

        @Override
        public void run() {
            mHavePendingVsync = false;
            // 3
            doFrame(mTimestampNanos, mFrame);
        }
    }
```
这个FrameDisplayEventReceiver就有点厉害了，它继承于抽象类`DisplayEventReceiver`,实现了`Runnable`接口。主要是native层中检测到VSYNC信号的时候触发回调执行Java层中的函数，可以说是Java和Native中沟通的使者。它本身构造不进行操作，直接继承于父类`DisplayEventReceiver`。

``` java
    public DisplayEventReceiver(Looper looper, int vsyncSource) {
        if (looper == null) {
            throw new IllegalArgumentException("looper must not be null");
        }

        mMessageQueue = looper.getQueue();
        // 1
        mReceiverPtr = nativeInit(new WeakReference<DisplayEventReceiver>(this), mMessageQueue,
                vsyncSource);

        mCloseGuard.open("dispose");
    }

    // Called from native code.
    @SuppressWarnings("unused")
    private void dispatchVsync(long timestampNanos, int builtInDisplayId, int frame) {
        onVsync(timestampNanos, builtInDisplayId, frame);
    }
```
由注释1中知道，调用了函数`nativeInit()`并将本身及所属线程的消息队列`messageQueue`传入native层，然后返回一个long值。这一点和 MessageQueue 类中的初始化很像。此处将自身传入，native接收到VSYNC信号的时候就可以回调触发java层了。触发的时候会调用`dispatchVsync()`函数 -> `onVsync()`来触发。在构造函数中可以看到就会触发注释2，然后由于异步消息会优先处理执行本身的`run()`函数，最终触发`doFrame()`进行界面的绘制。  

除了接收VSYNC信号之外，它还可以主动申请等待VSYNC信号，当下一帧VSYNC信号到来时就会通过native层回调到java层触发`onVsynC()`。

``` java
    /**
     * Schedules a single vertical sync pulse to be delivered when the next
     * display frame begins.
     */
    public void scheduleVsync() {
        if (mReceiverPtr == 0) {
            Log.w(TAG, "Attempted to schedule a vertical sync pulse but the display event "
                    + "receiver has already been disposed.");
        } else {
            // 1
            nativeScheduleVsync(mReceiverPtr);
        }
    }
```
注释1便是通过native方法主动去申请等待VSYNC信号。当Vsync信号来的时候上述说的`dispatchVsync()`就会触发，最终触发`onFrame()`进行绘制。

接下来，我们来看下`Choreographer`的使用。  

### Choreographer的使用

在`ViewRootImpl.java`文件中，在`scheduleTraversals()`绘制方法中我们会看到调用`mChoreographer.postCallback(Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);`。其中，通过`postCallback()`来进行`Choreographer`的使用的，最终通过`postCallbackDelayedInternal()`来实现的。

``` java
    private void postCallbackDelayedInternal(int callbackType,
            Object action, Object token, long delayMillis) {
        if (DEBUG_FRAMES) {
            Log.d(TAG, "PostCallback: type=" + callbackType
                    + ", action=" + action + ", token=" + token
                    + ", delayMillis=" + delayMillis);
        }

        synchronized (mLock) {
            final long now = SystemClock.uptimeMillis();
            final long dueTime = now + delayMillis;
            // 1
            mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);

            if (dueTime <= now) {
                // 2
                scheduleFrameLocked(now);
            } else {
                // 3
                Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
                msg.arg1 = callbackType;
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, dueTime);
            }
        }
    }
```

其中参数`action`一般为Runnable，在注释1中插入任务到`callbackType`类型的任务数组`mCallbackQueues`中，如果当前触发时间小于当前时间(一般都为立即触发，所以`delayMills`都为0,`dueTime`都和now相等)，那么就会进入注释2调用`scheduleFrameLocked()`。如果想要将来某个时间触发那么就会加一个`delayMills`延迟，然后在注释3中采用异步Message插入到消息队列中，确保优先执行界面绘制信息，最后依旧会通过`FrameHandler`触发`scheduleFrameLocked()`。我们再来看下这个函数。

``` java
    private void scheduleFrameLocked(long now) {
        if (!mFrameScheduled) {
            mFrameScheduled = true;
            if (USE_VSYNC) {
                if (DEBUG_FRAMES) {
                    Log.d(TAG, "Scheduling next frame on vsync.");
                }

                // If running on the Looper thread, then schedule the vsync immediately,
                // otherwise post a message to schedule the vsync from the UI thread
                // as soon as possible.
                if (isRunningOnLooperThreadLocked()) {
                    // 1
                    scheduleVsyncLocked();
                } else {
                    Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_VSYNC);
                    msg.setAsynchronous(true);
                    mHandler.sendMessageAtFrontOfQueue(msg);
                }
            } else {
                final long nextFrameTime = Math.max(
                        mLastFrameTimeNanos / TimeUtils.NANOS_PER_MS + sFrameDelay, now);
                if (DEBUG_FRAMES) {
                    Log.d(TAG, "Scheduling next frame in " + (nextFrameTime - now) + " ms.");
                }
                // 2
                Message msg = mHandler.obtainMessage(MSG_DO_FRAME);
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, nextFrameTime);
            }
        }
    }
```
函数中先判断是否采用垂直同步，Android 4.1之后默认采用垂直同步。然后再判断当前线程是否为 Choreograper 所在线程，如果不是则通过 FrameHandler 类型的`mHandler`在 choreographer 线程最终调用`scheduleVsyncLocked()`等待垂直信号同步。而如果是，那么就直接进行注释1中`scheduleVsyncLocked()`。 当然，如果不是垂直同步，那么就通过异步消息直接调用`doFrame()`进行绘制。`scheduleVsyncLocked()`函数最终会调用`scheduleVsyncLocked()`。

``` java 
    private void scheduleVsyncLocked() {
        mDisplayEventReceiver.scheduleVsync();
    }

    ** DisplayEventReceiver.java **
    public void scheduleVsync() {
        if (mReceiverPtr == 0) {
            Log.w(TAG, "Attempted to schedule a vertical sync pulse but the display event "
                    + "receiver has already been disposed.");
        } else {
            // 1
            nativeScheduleVsync(mReceiverPtr);
        }
    }
```
从上面代码看到，最终调用父类 DisplayEventReceiver 中的`scheduleVsync()`方法，在注释1中调用native 方法进行等待VSYNC信号同步。而在native中一旦检测到VSYNC信号，那么就会触发父类 DisplayEventReceiver 的`dispatchVsync()`，从native中调用传递到Java层中。

``` java
    // Called from native code.
    @SuppressWarnings("unused")
    private void dispatchVsync(long timestampNanos, int builtInDisplayId, int frame) {
        onVsync(timestampNanos, builtInDisplayId, frame);
    }
```
而`onVsync()`方法则是在子类 FrameDisplayEventReceiver 中进行过重写。我们来看一下。

``` java
        @Override
        public void onVsync(long timestampNanos, int builtInDisplayId, int frame) {
            // ......

            mTimestampNanos = timestampNanos;
            mFrame = frame;
            Message msg = Message.obtain(mHandler, this);
            msg.setAsynchronous(true);
            mHandler.sendMessageAtTime(msg, timestampNanos / TimeUtils.NANOS_PER_MS);
        }
```
Message 中的 Runnable 为this，而 FrameDisplayEventReceiver 也是实现了 Runnable 接口，所以这里的异步消息会实现 FrameDisplayEventReceiver 中重写的 `run()` 函数。

``` java
        @Override
        public void run() {
            mHavePendingVsync = false;
            doFrame(mTimestampNanos, mFrame);
        }
```
重写的`run()`中实现了`doFrame()`方法。

``` java
    void doFrame(long frameTimeNanos, int frame) {
        final long startNanos;
        synchronized (mLock) {
            // ......
            if (jitterNanos >= mFrameIntervalNanos) {
                final long skippedFrames = jitterNanos / mFrameIntervalNanos;
                if (skippedFrames >= SKIPPED_FRAME_WARNING_LIMIT) {
                    // 1
                    Log.i(TAG, "Skipped " + skippedFrames + " frames!  "
                            + "The application may be doing too much work on its main thread.");
                }
                final long lastFrameOffset = jitterNanos % mFrameIntervalNanos;
                if (DEBUG_JANK) {
                    Log.d(TAG, "Missed vsync by " + (jitterNanos * 0.000001f) + " ms "
                            + "which is more than the frame interval of "
                            + (mFrameIntervalNanos * 0.000001f) + " ms!  "
                            + "Skipping " + skippedFrames + " frames and setting frame "
                            + "time to " + (lastFrameOffset * 0.000001f) + " ms in the past.");
                }
                frameTimeNanos = startNanos - lastFrameOffset;
            }
            // ......
        }

        try {
            // 2
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "Choreographer#doFrame");
            AnimationUtils.lockAnimationClock(frameTimeNanos / TimeUtils.NANOS_PER_MS);

            mFrameInfo.markInputHandlingStart();
            doCallbacks(Choreographer.CALLBACK_INPUT, frameTimeNanos);

            mFrameInfo.markAnimationsStart();
            doCallbacks(Choreographer.CALLBACK_ANIMATION, frameTimeNanos);

            mFrameInfo.markPerformTraversalsStart();
            doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);

            doCallbacks(Choreographer.CALLBACK_COMMIT, frameTimeNanos);
        } finally {
            AnimationUtils.unlockAnimationClock();
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }

        if (DEBUG_FRAMES) {
            final long endNanos = System.nanoTime();
            Log.d(TAG, "Frame " + frame + ": Finished, took "
                    + (endNanos - startNanos) * 0.000001f + " ms, latency "
                    + (startNanos - frameTimeNanos) * 0.000001f + " ms.");
        }
    }
```
调用的`doFrame()`主要有两个地方。注释1中就是我们丢帧之后的提示信息。注释2中，依次开始执行任务队列 `mCallbackQueues` 中不同任务类型的任务了。而我们之前在调用 Choreographer 中的`postCallback()`将任务按时间先后顺序插入了任务队列中，这时就开始执行。再回到 ViewRootImpl.java 文件。任务为 TraversalRunnable 。来看下它的具体方法。

``` java
    final class TraversalRunnable implements Runnable {
        @Override
        public void run() {
            doTraversal();
        }
    }
```
其中调用了`doTraversal()`，这个方法可能大家印象不是很深，但是这个方法中调用的`performTraversals()`方法，大家就很熟悉了。

``` java
    void doTraversal() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            // 1
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }

            // 2
            performTraversals();

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
```
注释1中移除了消息的同步屏障，原因很简单，因为现在已经执行了我想要优先执行的绘制动作的异步消息，所以这个时候应该移除同步屏障确保其他同步消息可以正常的执行。  

而注释2中调用的`performTraversals()`大家很熟悉，其中会依次执行`onLayout`, `onMeasure()`, `onDraw()`这些函数。进行了View的绘制。  


## 小结
这一章的介绍还比较片面的介绍了`Choreographer`的工作原理。没有介绍Android的渲染流程，三重缓存，为何使用VSYNC信号，以及Surface和SurfaceFlinger，以及使用匿名内存共享机制来进行进程间通信的。这些内容之后有时间会继续给大家补充上。