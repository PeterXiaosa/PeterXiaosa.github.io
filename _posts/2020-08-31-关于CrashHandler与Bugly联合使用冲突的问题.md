---
layout: post
title: 关于CrashHandler与Bugly联合使用冲突的问题
categories: [Android,Bugly]
description: 在使用CrashHandler和Bugly的时候，是否会出现冲突的问题，导致一个崩溃只被一个库采集的问题。
keywords: Android，Java, Bugly
---


# CrashHandler 与 Bugly 是否会产生冲突的问题

## 问题起因

上次和一个小伙伴技术交流的时候，刚好在交流崩溃信息收集这一块，我告诉他我采集开发和线上崩溃的错误的时候采用了我自定义的 CrashHandler 和 腾讯的 Bugly，一个把崩溃信息写文件记录在本地，一个将崩溃信息上传至服务器。然后他突然问了我一个问题直接把我给整懵逼了。当时他和我说，你使用 CrashHandler 和 Bugly 去采集错误信息，这两者都是用来采集崩溃日志的，那你这不会出现冲突吗？ 你是不是会出现 CrashHandler采集到了崩溃信息然后导致错误信息无法上传 Bugly，或者 Bugly上传结果本地无法采集到的问题。  

听到他提的这个问题，当时心里就是一顿无语。这有什么关系吗，我本地没看到错误信息我就去 Bugly 控制台看啊，我管它哪个采集到了总之我能找到我的崩溃信息不是。想着不好驳小伙伴面子，还是说了说这倒是没考虑过这个问题。今天想着问题还是需要解决，这样不明不白也不是那么回事。所以也就去看了看 CrashHandler 的处理以及 Bugly 的源码。

## 问题探究的过程  

### 崩溃信息采集 —— UncaughtExceptionHandler

首先，探究这个问题之前，我们需要了解一个接口 `UncaughtExceptionHandler`, 这个接口的作用是线程由于一个未捕获的异常突然终止的时候会触发这个接口的`uncaughtException()`方法, 这个方法会将这个线程和异常作为参数传递出去。也就是说如果我们需要采集崩溃信息，那么我们就需要使用这个接口。其实这个接口在 Android 也默认有过使用，在 `RuntimeInit.java`中。

* frameworks\base\core\java\com\android\internal\os\RuntimeInit.java
``` java
    protected static final void commonInit() {
        // ......

        /*
         * set handlers; these apply to all threads in the VM. Apps can replace
         * the default handler, but not the pre handler.
         */
        Thread.setUncaughtExceptionPreHandler(new LoggingHandler());
        // 1
        Thread.setDefaultUncaughtExceptionHandler(new KillApplicationHandler());

        // ......
    }
```
`commonInit()`方法在 RuntimeInit 的 `main()`中会调用。通过代码中的注释我们可以知道，在注释1处，我们设置了 DefaultUncaughtExceptionHandler，并且这个 Handler会应用到虚拟机的所有线程中。`killApplicationHandler` 这个类实现了 `UncaughtExceptionHandler` 接口，并且在 `uncaughtException()`方法中，将崩溃告知了 `AMS`,然后将进程杀死。所以说 Android 其实默认给所有线程设置了 `KillApplicationHandler` 这个 ExceptionHandler。

### 自定义崩溃信息采集器 —— CrashHandler
然后，我们再来看了下自定义的 CrashHandler的代码，这里就直接挑主要的放吧。

``` java
public class CrashHandler implements UncaughtExceptionHandler {

    public void init(Context context) {
        // 1  先获取默认的 ExceptionHandler
        mDefaultCrashHandler = Thread.getDefaultUncaughtExceptionHandler();
        // 2  然后把自定义的 ExceptionHandler 设置为线程默认的 ExceptionHandler
        Thread.setDefaultUncaughtExceptionHandler(this);
        mContext = context.getApplicationContext();
    }

    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        // 3  处理崩溃的时候自己需要处理的事情
        dumpExceptionToSDCard(ex);
        uploadExceptionToServer();
        ex.printStackTrace();

        if (mDefaultCrashHandler != null) {
            // 4  将崩溃信息传到线程之前的默认的 ExceptionHandler 中去
            mDefaultCrashHandler.uncaughtException(thread, ex);
        } else {
            Process.killProcess(Process.myPid());
        }

    }
}
```

其实 CrashHandler 采用的是一种代理模式，代理了接口 `UncaughtExceptionHandler`，首先获取线程默认的 ExceptionHandler, 然后将自己自定义的 ExceptionHandler 设置为线程默认的 ExceptionHandler。也即上述说的 `KillApplicationExceptionHandler`。  

在发生崩溃的时候，触发 `uncaughtException()`方法，注释3处在崩溃的时候做了两件事，首先将日志写成文件存在本地，另一件事就是上传错误信息到服务器。然后在注释4处，实现被代理对象的 `uncaughtException()`方法。


### Bugly错误信息采集

Bugly 的代码使用了混淆，是不开源的。所以使用了 jd-gui 来反编译，发现 Bugly 也是采用同样的方法来收集错误信息的。  

![Bugly代码](http://xiaosalovejie.top/images/bugly_code.png)   

### 采集冲突问题的分析  

所以说两个库都是采用 `Thread.setDefaultUncaughtExceptionHandler(this)` 来收集的，会不会一个替换了另外一个呢，导致只有一个库生效呢。  

从代码上分析的话是不会的，因为我们自定义的 `CrashHandler` 是采用代理模式，代理了系统默认的 `KillApplicationExceptionHandler`，然后 Bugly 中通过 `Thread.getDefaultUncaughtExceptionHandler()` 获取的便是我们的 CrashHandler 对象，然后 Bugly 又采用代理模式，代理了 `CrashHandler`, 所以说当崩溃发生的时候先触发 Bugly, 然后触发我们的 CrashHandler， 最后再触发系统默认的 `KillApplicationExceptionHandler`。 (如果先执行 CrashHandler 初始化，再进行 Bugly 的初始化， 反之同理)  

而网上有些文章资料说，需要先初始化自定义的 CrashHandler， 然后再初始化 Bugly，这样才不会出现采集冲突的问题，但其实是不对的，二者初始化的先后顺序并不会发生采集冲突的问题，因为都是采用的静态代理，代理了 `UncaughtExceptionHandler` 接口。  

虽然二者初始化顺序不会影响崩溃信息的采集，但是有一点需要注意。这里采用伪代码来说明。

``` java
    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        // 必须先处理自己的事务方法。
        doMyThing();
        if (mDefaultCrashHandler != null) {
            // 代理方法传递
            mDefaultCrashHandler.uncaughtException(thread, ex);
        } else {
            Process.killProcess(Process.myPid());
        }
        // 代码无法执行到此处
    }
```

在 `uncaughtException()` 方法中，我们要把自定义的方法放在前面执行，然后再执行代理方法。因为代码无法执行到代理方法后面了，如果自定义方法写在后面，那么将永远无法执行到。原因其实我们之前有提到过，因为 Android 系统默认给我们设置了 ExceptionHandler, 也即 `KillApplicationHandler`, 它的 `uncaughtException()` 方法中向 `AMS` 报告了应用 Crash 的情况，随后就将进程杀死。进程被杀，所以后面的代码也即无法执行到了。

### 对采集冲突的验证  

对代码分析之后，我们得到初始化顺序并不会对采集崩溃日志有影响的这个结论。那么我们可以来验证一下这个结论，验证其实很简单。  

我们首先在代码中定义一个异常 `MyException`，继承于 `RuntimeException`，然后抛出的错误信息可以为`tag1`类似这样带有标志的信息， 定义 `MyException` 是为了在 Bugly 控制台中好搜索，因为崩溃日志实在太多了= =， 这个时候可以先初始化 CrashHandler，然后再初始化 Bugly。运行程序出现崩溃，然后在 Bugly 控制台刷新界面就可以看到我们的错误日志了，并且同时在本地文件中也找到了我们这一次的崩溃日志。  

然后，我们改变初始化顺序，先初始化 Bugly， 然后再初始化 CrashHandler， 崩溃日志代码中修改为 `tag2`，然后运行。同样，我们在控制台和本地都又能找到这一次的崩溃日志。  

两次结果证明，初始化顺序并不会对采集崩溃日志有影响的这个结论是正确的。

## 小结

通过上述的分析我们可以知道两点。  

* 初始化顺序并不会对采集崩溃日志有影响，不会造成一个库采集到了而另外一个库没有采集到。  

* 采集崩溃信息时，由于系统默认的 ExceptionHandler 会杀死进程，所以我们自定义的方法需要放在 `mDefaultCrashHandler.uncaughtException()` 之前，否则会出现由于进程被杀死，而自定义方法无法执行到的问题。