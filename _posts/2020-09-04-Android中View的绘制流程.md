---
layout: post
title: Android中View的绘制流程
categories: [Android]
description: 介绍 Android 中 View，ViewGroup的绘制流程。
keywords: Android，View的绘制
---


# Android 中 View 的绘制流程

我们之前有简单介绍过 Choreographer 的工作原理。它接收 VSYNC 信号之后调用 `performTraversals()`进行界面的绘制。而该方法 `performTraversals()` 便是我们今天介绍的 View 绘制流程的入口方法。在 Android 中关于 View的比较重要且比较常见的有 View的绘制流程以及 View 事件触摸传递。对 View 的绘制流程熟悉之后，我们就会对自定义 View 有更深一步的了解。今天，我们就先来介绍一下 View 的绘制流程。  

View 的绘制流程主要分为三步，按照执行先后顺序的话分别是 `onMeasure()`,`onLayout()`和`onDraw()`。同时我们还需要注意一点，ViewGroup 是 View 的子类。好了，现在我们就先来了解一下`onMeasure()`吧。  

## onMeasure  
`ViewRootImpl.java`中的`performTraversals()` 会调用`measureHierarchy()`。先直接上代码吧。

``` java
    private boolean measureHierarchy(final View host, final WindowManager.LayoutParams lp,
            final Resources res, final int desiredWindowWidth, final int desiredWindowHeight) {
        int childWidthMeasureSpec;
        int childHeightMeasureSpec;
        boolean windowSizeMayChange = false;
        // ......

        boolean goodMeasure = false;

        if (!goodMeasure) {
            // 1
            childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
            childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
            // 2
            performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
            if (mWidth != host.getMeasuredWidth() || mHeight != host.getMeasuredHeight()) {
                windowSizeMayChange = true;
            }
        }
        // ......

        return windowSizeMayChange;
    }
```
先介绍一下`performTraversals()`调用该方法的参数值，`LayoutParams lp`为默认的`new WindowManager.LayoutParams()`，所以`lp`的宽高即为默认的`match_parent`。而`desiredWindowWidth`和`desiredWindowHeight`则为屏幕的宽高。所以在注释1处会通过`getRootMeasureSpec()`得到两个 MeasureSpec 值。关于 View.MeasureSpec，我们可以看下官方文档的解释。

```
 A MeasureSpec encapsulates the layout requirements passed from parent to child. Each MeasureSpec represents a requirement for either the width or the height. A MeasureSpec is comprised of a size and a mode. There are three possible modes:
```

概述下来就，MeasureSpec 是通过将布局参数值压缩之后从父View 传递给子View的。每一个 MeasureSpec 都代表了宽或者高。MeasureSpec 是通过尺寸大小（比如宽高值）和模式(例如 match_parent)组成的。一共有三种模式。  

所以注释1中得到的两个宽高由于参数我们可以知道，传入的参数都是屏幕的宽高，以及`LayoutParam` 都是`match_parent`。所以得到的为屏幕的宽高(因为ViewRootImpl中的View为DecoreView)。然后将这个宽高传入注释2中的`performMeasure()`函数。

``` java
    private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
        if (mView == null) {
            return;
        }
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "measure");
        try {
            mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
    }
```

内部调用了 View 的 measure()。并且将它的宽高，传入。

``` java
    /**
     * 这个方法调用为了知道一个View应该是多大。父View提供了宽和高的限制信息。
     * <p>
     * This is called to find out how big a view should be. The parent
     * supplies constraint information in the width and height parameters.
     * </p>
     * 一个View实际的测量工作表现在这个方法调用的onMeasure(int, int)方法中。因此，
     * 可以并且必须被子类调用onMeasure(int, int)方法
     * <p>
     * The actual measurement work of a view is performed in
     * {@link #onMeasure(int, int)}, called by this method. Therefore, only
     * {@link #onMeasure(int, int)} can and must be overridden by subclasses.
     * </p>
     *
     *
     * @param widthMeasureSpec Horizontal space requirements as imposed by the
     *        parent
     * @param heightMeasureSpec Vertical space requirements as imposed by the
     *        parent
     *
     * @see #onMeasure(int, int)
     */
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        // ......

        if (forceLayout || needsLayout) {
            // first clears the measured dimension flag
            mPrivateFlags &= ~PFLAG_MEASURED_DIMENSION_SET;

            resolveRtlPropertiesIfNeeded();

            int cacheIndex = forceLayout ? -1 : mMeasureCache.indexOfKey(key);
            if (cacheIndex < 0 || sIgnoreMeasureCache) {
                // measure ourselves, this should set the measured dimension flag back
                onMeasure(widthMeasureSpec, heightMeasureSpec);
                mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            } else {
                long value = mMeasureCache.valueAt(cacheIndex);
                // Casting a long to int drops the high 32 bits, no mask needed
                setMeasuredDimensionRaw((int) (value >> 32), (int) value);
                mPrivateFlags3 |= PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            }
            // ......
        }
        // ......
    }
```
View的 measure 方法的作用其实在它的注释中写的明明白白了，我也在注释中进行了大致的翻译。这个方法是为了测量出 View 有多宽多高。并且注意到 measure 这个方法是final的，所以我们无法在子类中重写。我们只能重写 `onMeasure()`，通过`onMeasure()`方法实际测量出 View 实际的宽高。接下来，我们来看下 `onMeasure()`是如何进行测量的。

``` java
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```

方法内部只调用了一个方法，但是注释有很多。由于篇幅太长，所以我去除来总结一下，就是由`measure()`触发的这个方法是为了测量View的宽高的，子类应该重写这个方法来得到更精确的宽高(因为子类有自己的内容)。当子类重写这个方法的时候需要确保View会大于设置的最小宽高，否则就会抛异常了。 

如果是 View 的话，那么执行完了`onMeasure()`方法后，它的`mMeasuredWidth`和`mMeasuredHeight`就会被赋值。如果调用`getMeasuredWidth()`和`getMeasuredHeight()`就能获取到值了。也就是说，这两个方法的调用时机需要在`onMeasure()`之后，否则无法获取到值。  

那如果该 View 是 ViewGroup 的话，那就又有点不同了。方法注释中说，子类需要重写该方法来得到更精确的值。但是我们如果去`ViewGroup.java`中找的话，我们会发现其中并没有重写该方法。其实也很好理解，因为不同的 ViewGroup，它的放置顺序，方向这些都不一样。在父类ViewGroup中没有办法将其统一。所以我们可以以 LinearLayout 为例子来分析一下。在 LinearLayout 中会发现重写了这个方法。

``` java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mOrientation == VERTICAL) {
            measureVertical(widthMeasureSpec, heightMeasureSpec);
        } else {
            measureHorizontal(widthMeasureSpec, heightMeasureSpec);
        }
    }
```
两种方向，我们分析一个即可。来看下使用比较多的`measureVertical()`吧。

``` java
    void measureVertical(int widthMeasureSpec, int heightMeasureSpec) {
        // ......
        final int count = getVirtualChildCount();

        final int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        final int heightMode = MeasureSpec.getMode(heightMeasureSpec);

        // ......

        // See how tall everyone is. Also remember max width.
        // 1
        for (int i = 0; i < count; ++i) {
            final View child = getVirtualChildAt(i);
            
            // ......

            final LayoutParams lp = (LayoutParams) child.getLayoutParams();

            totalWeight += lp.weight;

            final boolean useExcessSpace = lp.height == 0 && lp.weight > 0;
            if (heightMode == MeasureSpec.EXACTLY && useExcessSpace) {
                // ......
            } else {
                // ......
                // 2
                measureChildBeforeLayout(child, i, widthMeasureSpec, 0,
                        heightMeasureSpec, usedHeight);
                        
                // ......
            }

            // ......
        }

        // ......

        maxWidth += mPaddingLeft + mPaddingRight;

        // Check against our minimum width
        maxWidth = Math.max(maxWidth, getSuggestedMinimumWidth());

        setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
                heightSizeAndState);

        if (matchWidth) {
            forceUniformWidth(count, heightMeasureSpec);
        }
    }

    void measureChildBeforeLayout(View child, int childIndex,
            int widthMeasureSpec, int totalWidth, int heightMeasureSpec,
            int totalHeight) {
        measureChildWithMargins(child, widthMeasureSpec, totalWidth,
                heightMeasureSpec, totalHeight);
    }

    protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```
注释1处，通过循环遍历 ViewGoup 即 LinearLayout 中的所有子View，然后通过`measureChildBeforeLayout()`到`measureChildWithMargins()`，最后调用子View 的 `measure()`方法。测量完子View之后，自己的宽高也测量好了，通过`setMeasuredDimension()`设置。而子View的测量方法 `measure()`我们之前也介绍过，`measure()`方法中，子View通过重写`onMeasure()`来计算宽高。  

所以一整套的 ViewGroup 的 Measure 流程，我们可以使用流程图来显示。

![View的measure流程](http://xiaosalovejie.top/images/View_measure.png)  


## onLayout
`onLayout()`调用流程其实和`onMeasure()`很像，它是 View 绘制流程的第二步。 之前我们知道 ViewRootImpl 在 `performTraversals()`中先调用`onMeasure()`测量 View 的大小。测量完之后需要定位 View 的位置。也即会调用 `performLayout()`。

``` java
    private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
            int desiredWindowHeight) {
        // ......
        final View host = mView;
        // ......
        try {

            // 1
            host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());

            // ......
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
        // ......
    }
```
这个方法内部会调用 View 的 `layout()` 方法，传入的值是之前 `onMeasure()`测量后的 measureWidth 和 measureHeight。而我们知道 ViewRootImpl 中的 view 是 ViewGroup。所以我们可以来看一下 ViewGroup 是如何重写该方法的。  

``` java
    @Override
    public final void layout(int l, int t, int r, int b) {
        if (!mSuppressLayout && (mTransition == null || !mTransition.isChangingLayout())) {
            if (mTransition != null) {
                mTransition.layoutChange(this);
            }
            super.layout(l, t, r, b);
        } else {
            // record the fact that we noop'd it; request layout when transition finishes
            mLayoutCalledWhileSuppressed = true;
        }
    }
```

这个方法和 View 的 measure() 方法一样，都是 final 修饰，不可以被子类重写。所以子类 ViewGroup 的 layout() 方法执行的其实是父类 View 的 `layout()` 方法。我们再来看看 View 的 `layout()` 方法。

``` java
    /**
     * 为一个 View 和它所有的子View 赋值尺寸和定位。
     * Assign a size and position to a view and all of its
     * descendants

     * 这个方法是 布局机制(View 的绘制机制) 的第二阶段。（第一阶段是 measure 测量）
     * 在这个阶段，每个父View调用它所有的子View的layout方法来定位它们。
     * 通常使用 measure() 传递过来的 measure 参数来实现定位的。

     * <p>This is the second phase of the layout mechanism.
     * (The first is measuring). In this phase, each parent calls
     * layout on all of its children to position them.
     * This is typically done using the child measurements
     * that were stored in the measure pass().</p>
     *
     * 派生类(子类)不应重写这个方法。含有子View的派生类应该重写onLayout方法。在onLayout中
     * 它们应该调用它们每个子View的 layout 方法。

     * <p>Derived classes should not override this method.
     * Derived classes with children should override
     * onLayout. In that method, they should
     * call layout on each of their children.</p>
     *
     * @param l Left position, relative to parent
     * @param t Top position, relative to parent
     * @param r Right position, relative to parent
     * @param b Bottom position, relative to parent
     */
    @SuppressWarnings({"unchecked"})
    public void layout(int l, int t, int r, int b) {
        // ......

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        // 1
        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            // 2
            onLayout(changed, l, t, r, b);

            // ......
        }
        // ......
    }
```

View 中的这个 layout() 方法的注释解释了很多。具体可以看注释旁边的翻译。方法主要是为了定位子View位置的。带有子View的ViewGroup应该重写 `onLayout()`方法。然后在它们的子View中调用layout()。  

在注释1处，最终调用`setFrame()`函数。内部会将View的左上右下四个值依次赋值给mLeft,mTop,mRight,mBottom。所以执行完layout()方法，调用 getWidth(),getHeight() 才有效。在注释2中，就会调用 `onLayout()` 方法了。可是 View 中的 `onLayout()` 方法是空的，需要子View来重写。 这里，我们以 LinearLayout 这个 ViewGroup 来做示例。

代码太多而且逻辑和 `onMeasure()`有点类似，就不贴了。通过相对于父类的左上右下的位置，和自己本身的padding,margin 以及 layoutParams 这些参数，来定义子 View 的位置。依次调用 `setChildFrame()`来调用子View 的 `layout()`，然后再触发子 View 的 `onLayout()`。即子 View 根据父View 给允许它的左上右下位置值，以及它本身的LayoutParams属性值来确定子View本身的属性值。同时 View 绘制的第一步 Measure 流程就是为了定位子 View 的位置而执行准备的。

## onDraw()

`onDraw()`的调用流程和上述两个流程差不多，不再继续重复说明了。主要是 `performTravsersals()`中调用 `performDraw()`，然后调用 `draw()`，`drawSoftware()`,在 `drawSoftware()`中调用 `View.draw()`。在 `View.draw()`中会调用 `onDraw()`方法，同时会触发 ViewGroup 中重写的 `dispatchDraw()`，将子View进行绘制。其中绘制所用的画布 Canvas 则通过 Surface 可以获取到。

# 小结

* View 的绘制流程依次为 `onMeasure()`,`onLayout()`,`onDraw()`。 `onMeasure()`测量是为了后一步的定位 View 的位置做准备，而 `onLayout()`则是将各个View的位置确定好，知道每个View在相对于父类都是在什么位置。确定好后，则通过调用 `onDraw()`将它们绘制出来，这样就完成了 View的绘制。

* View 的测量 measure 和 定位 layout 都是通过父类的大小位置和本身的LayoutParams 以及padding，margin 以及方向等参数来确定的。


# 引用

* [Android应用层View绘制流程与源码分析](https://blog.csdn.net/yanbober/article/details/46117397)