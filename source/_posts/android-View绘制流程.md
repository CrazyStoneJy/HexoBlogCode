---
title: android View流程
date: 2018-01-30 17:02:01
tags: [android,view]
categories: [android]
---

android View的执行流程分析.初始的入口是从ViewRootImpl的performTraversals()方法开始执行,接下来开始分析这个方法.由于这个方法有上千行代码,咱们是粗略的分析一下其中的measure(),layout(),draw()方法.

当每次调用requestLayout()是就会调用performTraversals()方法来执行绘制流程.

1. **Measure阶段**

perfromTraversal()中调用performMeasure()方法,在调用performMeasure()之前,先是粗略的计算了子view的大小,代码如下:

```java
int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
```
可以看到计算子View大小的方法是getRootMeasureSpec()方法,好,我们再看看这个方法的实现,

```java
 private static int getRootMeasureSpec(int windowSize, int rootDimension) {
        int measureSpec;
        switch (rootDimension) {

        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
        }
        return measureSpec;
    }
```

<!-- more -->

可以看到这里通过对rootDimension的大小进行分情况计算,具体的计算是调用了View类中的一个静态类MeasureSpec的静态方法,在看看这个方法.

```java
 public static int makeMeasureSpec(@IntRange(from = 0, to = (1 << MeasureSpec.MODE_SHIFT) - 1) int size,@MeasureSpecMode int mode) {
            // 这个boolean变量是判断当前android版本是否大于17
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                // android版本大于17,则执行下面这个算法.
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }
```

sUseBrokenMakeMeasureSpec是判断当前android是否大于17的boolean变量,如果小于17返回的measurespec的值为 size+mode,这块没看懂因为我没看过android17版本这块的源码,但是分析一下,size和mode相加以后,值混淆了,怎么再次获取size和mode的对应的值呢,没搞懂???

我们还是分析大于android17的情况吧,大于android17代码实现是这样的(size & ~MODE_MASK) | (mode & MODE_MASK),这个MODE_MASK是什么??我们看源码知道MODE_MASK是

```java
private static final int MODE_SHIFT = 30;
private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
public static final int UNSPECIFIED = 0 << MODE_SHIFT;
public static final int EXACTLY     = 1 << MODE_SHIFT;
public static final int AT_MOST     = 2 << MODE_SHIFT;
```

可以看到MODE_MASK是把3左移了30位,为什么这样做呢?好,我们继续分析,我们知道int类型它占32个bit,这里android把view的大小和分类使用了一个int值存储,高两位表示mode,低三十位表示size.因为mode类型就三种,使用2个bit就够了.好了我们在回到之前measurespec的计算makeMeasureSpec()方法中传来size和mode两个参数,计算size使用size & ~MODE_MASK,MODE_SIZE是高两位为1,其余为0,~MODE_SIZE则为高两位为0,其余位为1,然后与size做&运算,计算结果还是size基本上还是本身因为size最大为屏幕的宽高最大的像素值肯定小于2的30次方-1.mode的计算就是和size是一样都是&运算.当两个值运算完以后再执行|运算,这样就将size和mode合在同一个int值中了.

当把childWithMeasureSpec和childHeightMeasureSpec计算完成后,传入performMeasure()方法中,performMeasure()方法其实就是调用view的measure()方法.

```java
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

接下来看看view的measure()方法.

```java
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
```

我们看到在measure()方法中有调用onMeasure()方法,注释也写的比较清楚,onMeasure()方法就是让view去自我测量.measure()方法中测逻辑现在真看不懂,但是我们现在先了解它的流程.知道关键方法调用即可.

接下来我们便看看我们都熟悉的onMeasure()方法中是干了什么.

```java
 protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```

我们看到onMeasure()方法中调用了一个setMeasureDimension()方法,这个方法是保存view测量得的宽高,必须在onMeasure()中调用该方法,如果没有调用会抛异常.

我们看看getDefaultSize()方法的实现:

```java
 public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
```

MeasureSpec.AT_MOST代表的是wrap_content,MeasureSpec.EXACTLY代表的是match_parent.

setMeasureDimension()方法最后的实现是:

```java
private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
        mMeasuredWidth = measuredWidth;
        mMeasuredHeight = measuredHeight;

        mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
    }
```

就是将计算的宽高赋值给了mMeasureWidth,mMeasureHeight.也就是当调用完setMeasureDimension()方法后,就可以调用getMeasureHeight()方法获取测量后的高,或者宽了.

```java
// 用来过滤mode
public static final int MEASURED_SIZE_MASK = 0x00ffffff;
public final int getMeasuredHeight() {
        return mMeasuredHeight & MEASURED_SIZE_MASK;
    }
```

我们在回到performTraversals()方法,我们通过源码可以看到在调用完performMeasure()方法后,又再次调用了performMeasure()方法,进行了2次测量.

```java
if (measureAgain) {
    if (DEBUG_LAYOUT) Log.v(mTag,
            "And hey let's measure once more: width=" + width
            + " height=" + height);
    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```
以上就是measure阶段的大体流程.

另外提一下ViewGroup的measure,ViewGroup是View的子类,并且是个abstract方法,他内部没有实现onMeasure()方法,为什么呢?因为onMeasure()方法是对view自身的测量,但是每个ViewGroup的自身大小是依据内部的子view的,而且每个ViewGroup的排布方式不一样,因此没有实现onMeasure()方法,需要让开发者在自定义ViewGroup重写该方法.

ViewGroup的onMeasure()方法是要测量每个子view的大小和自己本身的大小.

ViewGroup内部实现了一个叫measureChildren()的方法,用来测量子view的大小.

```java
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }
```

measureChild()方法是详细的测量每个child view实现.

```java
  protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

其中getChildMeasureSpec()这个方法很有参考价值,在计算child view的大小时.

```java
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```


2. **layout阶段**

layout主要是摆放child view的位置,在测量阶段完成后,performTraversals()方法会调用performLayout()方法进行计算子view的位置,在performLayout()方法中又调用了view的layout()方法,我们通过源码可以看到:

```java
 host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
```

接下来我们看看view的layout()方法,代码如下:

```java
public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);

            if (shouldDrawRoundScrollbar()) {
                if(mRoundScrollbarRenderer == null) {
                    mRoundScrollbarRenderer = new RoundScrollbarRenderer(this);
                }
            } else {
                mRoundScrollbarRenderer = null;
            }

            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
                }
            }
        }

        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;

        if ((mPrivateFlags3 & PFLAG3_NOTIFY_AUTOFILL_ENTER_ON_LAYOUT) != 0) {
            mPrivateFlags3 &= ~PFLAG3_NOTIFY_AUTOFILL_ENTER_ON_LAYOUT;
            notifyEnterOrExitForAutoFillIfNeeded(true);
        }
    }
```

可以看到在调用了onMeasure()方法,因此,了解了view的onMeasure()方法会多次的调用.

那我们在看看onLayout()方法,

```java
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    }
```

wtf居然是个空方法??恩,是这样的.因为onLayout()方法是用来排布子view的,view并没有子view.好吧,那我们看看ViewGroup的onLayout()方法.

```java
    @Override
    protected abstract void onLayout(boolean changed,
            int l, int t, int r, int b);
```

恩,这是一个抽象方法.因为ViewGroup是个抽象方法,所以可以有抽象方法,这个就是为什么我们在继承ViewGroup时,必须实现此方法的原因.因为每个ViewGroup的child view排布不尽相同.

3. **draw阶段**

在执行完measure和layout阶段后,会调用performDraw()方法,进行绘制.

```java
draw(fullRedrawNeeded);
```

具体方法调用看下面这张图即可:

![performDraw到draw的流程](http://or5n6ccgu.bkt.clouddn.com/18-2-1/82620731.jpg)

在这主要讲一下dispatchDraw()这个方法,这个方法主要就是用来画子view的,就是循环调用child view的onDraw()方法.恩,我们看看View这个类的disptach()方法.

```java
 protected void dispatchDraw(Canvas canvas) {

    }
```

恩,又是个空方法,因为view本身并没有子view所以该方法是个空方法,那我们来看看ViewGroup的dispatchView()方法.由于该方法比较长,我们只看关键步骤.

```java
 // Draw any disappearing views that have animations
        if (mDisappearingChildren != null) {
            final ArrayList<View> disappearingChildren = mDisappearingChildren;
            final int disappearingCount = disappearingChildren.size() - 1;
            // Go backwards -- we may delete as animations finish
            for (int i = disappearingCount; i >= 0; i--) {
                final View child = disappearingChildren.get(i);
                more |= drawChild(canvas, child, drawingTime);
            }
        }
```

我们看到了这是将可见的view的循环遍历去draw child view.那么接下在看看drawChild()这个方法.

```java
 protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
        return child.draw(canvas, this, drawingTime);
    }
```

恩,子view调用了draw方法,进行了自我绘制.

因此,通过以上分析,如果想要在ViewGroup的子view绘制之后,再搞点事情,可以在dispatchDraw()方法中,先调用super.dispatchDraw()方法后,在编写自定义的绘制.因为,ViewGroup的onDraw()方法先执行,后执行child view的onDraw(),child view的绘制会挡住parent view的绘制.所以,在super.dispatchDraw()方法后,等所有child view的都onDraw()执行完后,再绘制一些东西,就不会在遮住了.


### 总结

以上就是view整个绘制流程的大体步骤,在总结一下,总共有3个步骤:measure->layout->draw.

measure阶段:ViewRootImpl.performMeasure()->View.measure()->View.onMeasure(),如果是ViewGroup调用在onMeasure()方法时,会先去测量他的child view,测量完之后,再调用setMeasureDimension()保存最后的measuredWidth,measuredHeight.

layout阶段:ViewRootImpl.performLayout()->View.layout()->View.onLayout(),注意class View中onLayout是个空方法,因为onLayout是排布child view的,而View并没有child view.class ViewGroup中onLayout是个abstract方法,因为每个ViewGroup的child view排布都不一样,索引需要继承ViewGroup的类自己去重写.

draw阶段:ViewRootImpl.performDraw()->View.draw()->View.onDraw().这里只强调一下dispatchDraw()方法,该方法是绘制child view的.在onDraw()方法之后被调用了.

用张图说明view最简流程:

![view简约流程](http://or5n6ccgu.bkt.clouddn.com/18-2-1/36470492.jpg)


### 参考
<android开发艺术探索>
