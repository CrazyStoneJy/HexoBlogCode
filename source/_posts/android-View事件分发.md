---
title: android View事件分发
date: 2018-03-01 14:10:53
tags: [android,view]
categories: [android]
---

#### view事件流程分析

view的事件流程我们先从activity说起,当一个view被点击时,其实事件消息是从activity的dispatchTouchEvent开始进行传递,我们走进activity的dispatchTouchEvent看看究竟是怎么实现的.

```java
  public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
```
<!--  more -->

我们看到activity的dispatchTouchEvent是调用了Window的superDispatchTouchEvent方法,那我们在看看Window的superDispatchTouchEvent,我们都知道Window是个抽象类,并且仅有一个子类即PhoneWindow,我们边看看PhoneWindow的
那我们在看看Window的superDispatchTouchEvent方法.在API26的源码中无法找到PhoneWindow的源码,因此这块代码借鉴Android艺术开发探索的View事件机制部分代码.

> Abstract base class for a top-level window look and behavior policy. An instance of this class should be used as the top-level view added to the window manager. It provides standard UI policies such as a background, title area, default key processing, etc.

The only existing implementation of this abstract class is android.view.PhoneWindow, which you should instantiate when needing a Window.

```java
public boolean superDispatchTouchEvent(MotionEvent event){
    return mDecor.superDispatchTouchEvent(event);
}
```

我们看到事件又传递到了DecorView中,我们知道DecorView是继承自FrameLayout,而我们在调用setContentView时是将View直接添加到了DecorView的第一个位置.好了不多说了,DecorView将事件又传递到了setContentView方法中的最外层的ViewGroup中,我们便看看ViewGroup中的dispatchTouchEvent方法.下面伪代码显示:

```java
public boolean dispatchTouchEvent(MotionEvent event) {
    boolean consume = false;
    if (onInterceptTouchEvent(event)){
        consume = onTouchEvent(event);
    }else{
        // View[] childs = getChildren();
        // for (View child : childs) {
        consume = child.dispatchTouchEvent(event);
        // }
    }
    return consume;
}
```
可以看到先是判断拦截,如果没有拦截,则将事件分发给子View进行处理.

#### 总结

android事件传递伪代码. 这个是ViewGroup中事件分发的伪代码.

```java
public boolean dispatchTouchEvent(MotionEvent event) {
    boolean consume = false;
    if (onInterceptTouchEvent(event)){
        consume = onTouchEvent(event);
    }else{
        // View[] childs = getChildren();
        // for (View child : childs) {
        consume = child.dispatchTouchEvent(event);
        // }
    }
    return consume;
}
```


#### view滑动冲突处理

1. 外部拦截法

外部拦截发主要是通过父容器进行拦截,调用父view的onInterceptTouchEvent()方法,来通过不同的手势操作进行对子view事件的拦截.以下是代码实现.

```java
public boolean onInterceptTouchEvent(MotionEvent event) {
    boolean intercept = false;
    int x = (int) event.getX();
    int y = (int) event.getY();
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            intercept = false;            
            break;
        case MotionEvent.ACTION_MOVE:
            if(calOffset()){
                intercept = true;
            }else{
                intercept = false;
            }
            break;
        case MotionEvent.ACTION_UP:
            intercept = false;
            break;
    }
    return intercept;
}
```
其中,calOffset是开发者计算,什么时候需要拦截,什么时候不需要拦截.

2. 内部拦截法

内部拦截法是指把所有事件都传给子view处理,如果子view需要消费,就消费掉,否则,由父view处理.
具体伪代码实现.

子view的dispatchTouchEvent
```java
public boolean dispatchTouchEvent(MotionEvent event) {
    int x = (int) event.getX();
    int y = (int) event.getY();
    switch(event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            // parent view 不拦截
            parent.rquestDisallowInterceptTouchEvent(true);
            break;
        case MotionEvent.ACTION_UP:
        case MotionEvent.ACTION_CANCEL:
            parent.rquestDisallowInterceptTouchEvent(true);
            break;
        case MotionEvent.ACTION_MOVE:
            if(calOffest()) {
                parent.rquestDisallowInterceptTouchEvent(false);
            }
            break;
    }
    return super.dispatchTouchEvent(event);
}
```
在依据开发者写的calOffset方法,判断是否应该拦截.requestDisallowInterceptTouchEvent()方法的意思是:是否要取消父view的拦截,true为取消拦截,false反之.



#### 参考资料

Android开发艺术探索