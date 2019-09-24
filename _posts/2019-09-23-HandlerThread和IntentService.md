---
layout: post
title: HandlerThread和IntentService源码分析
categories: Android
description: 分析HandlerThread和IntentService源码，了解工作原理及使用场景。
keywords: Android、Handler、Looper、HandlerThread、IntentService
---

关于Android消息机制的原理及Handler，MessageQueue，Looper的原理我们在之前有分析过，而HandlerThread则是Android对线程加入了Handler进行了封装处理的产物。而IntentService则是在Service基础上加入了HandlerThread的封装，今天就它们我们来分析下源码及各自使用的场景。

## HandlerThread
HandlerThread是一种自带Looper的线程，`Looper`在创建线程的时候就会自动产生，HandlerThread自己会维护自己的一个消息队列。通过HandlerThread中的`Looper`创建出`Handler`，通过这个`Handler`就可以向HandlerThread中发送要执行的任务，因为创建的HandlerThread为非主线程（子线程），所以执行的是一种异步任务。同时因为HandlerThread拥有自己的消息队列，发送的任务也会被依次添加到该消息队列中，所以HandlerThread同时是顺序的执行异步任务。下面就以上特点，我们通过源码分析来验证。  

HandlerThread源码很简单，我们可以通过使用它的顺序来分析。首先使用的时候需要先调用`start()`启动线程，然后通过`Looper`构造`Handler`，然后通过`Handler`发送任务到HandlerThread中。而线程调用`start()`创建线程之后会调用`run()`执行，所以我们首先来看下`run()`函数。


```java
    @Override
    public void run() {
        mTid = Process.myTid();
        // 首先创建HandlerThread线程的Looper
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        // 设置线程优先级
        Process.setThreadPriority(mPriority);
        // Looper启动前的准备工作
        onLooperPrepared();
        // 开始Looper循环。
        Looper.loop();
        mTid = -1;
    }
```

在`run()`方法中，首先通过`Looper.prepare()`来创建Looper，然后设置完线程的优先级之后及Looper启动前的准备工作之后就调用`Looper.loop()`开始启动`Looper`处理消息队列中的消息了。这里的`onLooperPrepared()`是一个保护函数，可重写它来完成一些`loop()`前的操作。  

在消息队列启动之后，我们就可以通过线程中的`Looper()`来创建`Handler`了，在Android消息机制中我们有分析知道，`Handler`工作在其创建它的线程中，所以`Handler`是在HandlerThread中处理消息。而HandlerThread又是一个子线程,同时该子线程中也创建了`Looper`并执行了`loop()`，所以印证了开头所说，HandlerThread是一种顺序执行(串行执行)异步任务的Thread。所以如果某一个任务执行时间过长，那么就会导致之后的任务相对应的被延迟。  

最后，需要注意的一点就是，当HandlerThread使用完之后，因为子线程中的`Looper`一直不断地在`loop()`，所以我们需要及时的使用`quit()`或者`quitSafely()`进行退出。两者区别是`quit()`将会立即退出消息队列，不会再处理消息队列中的延迟消息(`postDelay`发送的消息)和非延迟消息，而`quitSafely()`则是会处理消息队列中的消息，然后再退出消息队列。


## IntentService
IntentService是一个抽象类，使用时需要自定义一个类来继承它。它继承于Service，并且内部自带`HandlerThread`，用于在Service中处理异步任务。自定义IntentService时，需要重写`onHandleIntent()`，该方法在`HandlerThread`(子线程)中执行。然后通过`startService()`来启动服务。我们先来看下IntentService创建时执行的`onCreate()`方法。

```java
    @Override
    public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.

        super.onCreate();
        // 创建HandlerThread。
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        // 然后启动HandlerThread线程。
        thread.start();

        mServiceLooper = thread.getLooper();
        // 创建Handler，用来将数据发送到HandlerThread线程中的任务执行。
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }
```
在创建IntentService时，会创建`HandlerThread`并启动，同时会根据`HandlerThread`的`looper`来创建一个类内部定义的Handler，这个在`HandlerThread`线程中执行的Handler是怎么处理任务的呢，我们再来看下类内部定义的`ServiceHandler`。

```java
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            // 调用重写的抽象函数来执行任务。
            onHandleIntent((Intent)msg.obj);
            // 执行完任务之后将服务关闭，无需用户手动关闭
            stopSelf(msg.arg1);
        }
    }
    ......
    // 需要重写的抽象函数，在这个函数中执行需要处理的任务
    protected abstract void onHandleIntent(@Nullable Intent intent);
```

`ServiceHandler`接收到消息时会先调用`IntentService`必须要重写的函数`onHandleIntent()`来执行任务，执行完任务之后便会调用`stopSelf()`来停止服务，无需用户手动关闭。  

知道了`IntentService`是如何来执行任务的，现在我们需要了解哪里触发了`ServiceHandler`来发送消息。在`Service`中我们知道当我们调用了`startService()`之后便会触发服务的`onStartCommand()`。同理，我们来`IntentService`的`onStartCommand()`函数。

```java
    /** 注释中也要求你使用IntentService时候需要重写 onHandleIntent()方法。
     * You should not override this method for your IntentService. Instead,
     * override {@link #onHandleIntent}, which the system calls when the IntentService
     * receives a start request.
     * @see android.app.Service#onStartCommand
     */
    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }
```

`onStartCommand()`函数中将我们传入的`Intent`传入了`onStart()`函数。

```java
    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        // ServiceHandler发送消息去执行任务
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }
```

可以看到，当调用`startService()`的时候触发`onStartCommand()`，然后在`onStart()`中ServiceHandler类型的handler会将`Intent`封装到message中发送，触发之前ServiceHandler中的`handleMessage()`,最终触发`onHandleIntent()`来执行任务。  

所以，使用`IntentService`时需要先自定义一个类继承它，然后重写抽象函数`onHandleIntent()`(任务逻辑在其中执行)，通过调用`startService`来触发`ServiceHandler`类型的handler发送消息给`HandlerThread`中的消息队列，所以`onHandleIntent()`会在子线程`HandlerThread`中执行，在任务执行完之后会自动调用`stopSelf()`停止服务。

## 实现Demo
实现demo  [GitHub传送门](https://github.com/PeterXiaosa/practiceProject/tree/master/app/src/main/java/com/peter/practiceproject/studyitem/handlerthread)

## 总结

* `HandlerThread`通过继承`Thread`，在子线程中会创建`Looper`并开启自己线程的消息队列，通过handler将任务发送到`HandlerThread`在子线程中执行，并且由于维护了自己的消息队列，所以执行任务是串行执行的。使用`HandlerThread`可以避免多开线程来执行多个任务，可通过handler不断发送任务到`HandlerThread`中执行。所以如果有任务耗时过长，就会导致后续任务执行的延迟。

* `IntentService`继承于Service，内部含有`HandlerThread`来处理任务。Service本身执行在主线程中，由于自带了`HandlerThread`，所以任务执行在子线程中，并且可以通过多次调用`startService`来多次执行任务。并且`IntentService`在每次执行完任务之后会自动停止服务，所以也无需用户手动停止。