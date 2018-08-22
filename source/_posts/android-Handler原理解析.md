---
title: android Handler原理解析
date: 2018-02-28 18:54:39
tags: [android,handler]
categories: [android]
---

Handler大家都知道可以来用线程间的通信,从一个worker thread处理完任务以后,可以通过handler发送message,切回到主线程来进行更新UI.

大家都知道Activity的入口在ActivityThread的main方法中.我们看看main函数中的核心实现.

```java
 public static void main(String[] args) {
        
        ...

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        ...

    }

```

<!-- more -->

1. Looper.prepare()

可以看到先执行了Looper.prepareMainLooper(),我们看看prepareMainLooper()实现

```java
public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
```

恩,我们发现prepareMainLooper()实际是调用的prepare()方法,那我们再看看prepare()内部实现.

```java
private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

这里是用了一个ThreadLocal.

> ThreadLocal： 线程本地存储区（Thread Local Storage，简称为TLS），每个线程都有自己的私有的本地存储区域，不同线程之间彼此不能访问对方的TLS区域。这里线程自己的本地存储区域存放是线程自己的Looper。

sThreadLocal.set(new Looper(quitAllowed));可以看到这是将Looper存储在ThreadLocal的ThreadMap中了,我们再看看Looper的构造方法.

```java
private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```

可以看到这里new了一个MessageQueue(),至此看来,Looper.prepare()主要是创建了一个MessageQueue.


2. Looper.loop()

我们再看看loop()方法,内部实现.剔除不相关的代码,只看核心方法.

```java
    public static void loop() {
        final Looper me = myLooper();
        
        final MessageQueue queue = me.mQueue;

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            ... 

            try {
                msg.target.dispatchMessage(msg);
                end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }

            ...

            msg.recycleUnchecked();
        }
    }
```

可以看到这个名叫Loopr的方法,果真是个死循环的方法.`queue.next()`可以知道这个死循环中,一直在轮巡`MessageQueue`中的`Message`,直到`MessageQueue`中的`Message`为空才退出该循环.
再看`msg.target.dispatchMessage(msg)`这个方法,msg的target是`Handler`对象,所以我们再看看`Handler`类中的`dispatchMessage(msg)`方法.

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
```

`dispatchMessage`方法主要就是分发处理`Message`,如果callback为空,就会调用我们所熟知的`handleMessage`方法.


至此看来,我们就通过Loopr.prepare(),Looper.loop(),就将该thread变为了LooperThread,这个线程就变成了一个能够处理message的线程.

#### send Message

因为我们主线程就是一个Looper线程,因此它管理着一个消息队列,我们可以自定义Handler来进行sendMessage,接下来我们在分析分析Handler的sendMessage方法.看源码我们知道了sendMessage,sendMessageDelay,post方法等,都是调用了`sendMessageAtTime`方法.

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

我们在分析分析`enqueueMessage`方法,

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

通过源码我们可知,如果mAsynchronous为true,说明是异步,那就执行异步的操作.否则,调用`MessageQueue`的`enqueueMessage`方法.

```java
boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
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

通过源码我们可以了解到通过比较消息延迟的大小,来不断将新加入的`Message`将入到原来的`Message`的next对象中,这个就是一个链表的结构,最终我们就将源源不断的消息形成了一个链式结构.以便在Looper.loop方法中循环遍历`Message`,至此,Handler,Looper,Message的原理基本分析完毕.

#### 总结

好了,我们在这里总结一下,UI线程本身就是一个Looper线程,它是管理这一个MessageQueue,使用Handler发送Message是将Message加入到消息队列中.loop方法一直在轮巡处理Message.为什么在worker thread中使用Handler去sendMessage,就会回到主线程,是因为,这个消息队列在主线程中,消息队列最后分发出的回调方法也在主线程中,这样就实现了在worker线程中操作耗时任务,使用Handler在主线程中更新的操作.



#### 参考资料

[Android 消息处理机制（Looper、Handler、MessageQueue,Message）](https://www.jianshu.com/p/02962454adf7)
