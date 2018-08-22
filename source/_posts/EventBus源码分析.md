---
title: EventBus源码分析
date: 2018-03-05 12:59:00
tags: [android,eventbus,源码分析]
categories: [android,源码分析]
---

EventBus我们都很熟悉了,它主要是通过注解+观察者模式+反射实现的.接下来我们通过它的源码来分析分析.
我们在使用EventBus时,先是注册,将当前类注册事件,`EventBus.getDefault().register(this)`,然后在定义一个有`@Subscriber`注解的方法来接受分发者传来的事件.好,首先我们来看看register()的实现.

#### register方法实现

以下是register的方法实现.

```java
 public void register(Object subscriber) {
        Class<?> subscriberClass = subscriber.getClass();
        // 获取订阅者中有Subscriber注解的方法
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
        synchronized (this) {
            for (SubscriberMethod subscriberMethod : subscriberMethods) {
                // 实现订阅
                subscribe(subscriber, subscriberMethod);
            }
        }
    }
```

我们看到`register`方法中,首先是通过调用`subscriberMethodFinder`中的`findSubscriberMethods`方法来获取有`Subscriber`注解的方法,那么接下来我们在看看这个方法的实现.

<!--  more -->

```java
 List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
        List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
        if (subscriberMethods != null) {
            return subscriberMethods;
        }

        if (ignoreGeneratedIndex) {
            subscriberMethods = findUsingReflection(subscriberClass);
        } else {
            subscriberMethods = findUsingInfo(subscriberClass);
        }
        if (subscriberMethods.isEmpty()) {
            throw new EventBusException("Subscriber " + subscriberClass
                    + " and its super classes have no public methods with the @Subscribe annotation");
        } else {
            METHOD_CACHE.put(subscriberClass, subscriberMethods);
            return subscriberMethods;
        }
    }
```

这一步主要是先在`METHOD_CACHE`中寻找是否有对应subscriberClass类的`@Subscriber`方法,如果有则返回该值,否则根据`ignoreGeneratedIndex`的值去按不同的方式的去查找具有`Subscriber`注解的方法.如果该类中没有`Subscriber`注解的方法,则将会抛出`EventBusException`异常,否则将该类和具有`Subscriber`注解的方法加入到`METHOD_CACHE`的键值对中.

在`ignoreGeneratedIndex`为true时,是通过反射去查找具有`Subscriber`注解的方法,接下来我们先看看该方法.

1. **findSubscriberMethods**

1.1 **findUsingReflection**

```java
private List<SubscriberMethod> findUsingReflection(Class<?> subscriberClass) {
        FindState findState = prepareFindState();
        findState.initForSubscriber(subscriberClass);
        while (findState.clazz != null) {
            findUsingReflectionInSingleClass(findState);
            findState.moveToSuperclass();
        }
        return getMethodsAndRelease(findState);
    }
```

先从`FindState`数组中获取一个`FindState`对象,在调用`initForSubscriber`方法,`initForSubscriber`方法中没有什么其他操作,就是简单赋值,接下来是个循环直到`findState`的`clazz`的属性为null为止,在循环体中调用了`findUsingReflectionInSingleClass`方法,那我们再看看该方法是用来做什么的.

```java
 private void findUsingReflectionInSingleClass(FindState findState) {
        Method[] methods;
        try {
            // This is faster than getMethods, especially when subscribers are fat classes like Activities
            // findState的clazz属性就是初始化时被赋的值,也就是调用register方法所在的类
            // 使用getDeclaredMethods是获取该类声明的方法(包括public,private,protected,friendly)不包//括父类和接口实现的方法.
            methods = findState.clazz.getDeclaredMethods();
        } catch (Throwable th) {
            // Workaround for java.lang.NoClassDefFoundError, see https://github.com/greenrobot/EventBus/issues/149
            methods = findState.clazz.getMethods();
            findState.skipSuperClasses = true;
        }
        for (Method method : methods) {
            int modifiers = method.getModifiers();
            // 判断该方法是否是public
            if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
                Class<?>[] parameterTypes = method.getParameterTypes();
                if (parameterTypes.length == 1) {
                    Subscribe subscribeAnnotation = method.getAnnotation(Subscribe.class);
                    if (subscribeAnnotation != null) {
                        // 获取有Subscriber注解的方法的参数的类型
                        Class<?> eventType = parameterTypes[0];
                        if (findState.checkAdd(method, eventType)) {
                            ThreadMode threadMode = subscribeAnnotation.threadMode();
                            // 将注解中的参数封装为SubscriberMethod对象,加入到subscriberMethods中
                            findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode,
                                    subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                        }
                    }
                } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                    // 如果有Subscriber注解的方法的参数不是一个则抛出异常
                    String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                    throw new EventBusException("@Subscribe method " + methodName +
                            "must have exactly 1 parameter but has " + parameterTypes.length);
                }
            } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                // 如果该方法中有Subscriber注解但是该方法不是public的则抛出异常
                String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                throw new EventBusException(methodName +
                        " is a illegal @Subscribe method: must be public, non-static, and non-abstract");
            }
        }
    }
```

该方法稍微有一些长,但是逻辑比较清晰,我画了张流程图,大家看一下.

![](http://or5n6ccgu.bkt.clouddn.com/18-3-5/19441680.jpg)

1.2 **findUsingInfo**

这个方法目前没研究明白,等研究透彻再给补上吧.


2. **subscriber**

上面通过反射获取到该类及其父类中的含有Subscriber注解的方法.接下来要做的事情就是将这些方法进行订阅.如何订阅则查看`subscriber`方法源码的实现.

```java
 private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
        // eventType我们通过前面的操作知道是具有Subscriber的方法的参数参数类型,
        // 即我们编写的MessageEvent等传递对象.
        Class<?> eventType = subscriberMethod.eventType;
        // 将订阅的类与具有Subscriber注解的方法封装为Subscription对象.
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
        // eventType表示方法中的参数的类型
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            if (subscriptions.contains(newSubscription)) {
                throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                        + eventType);
            }
        }

        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {
            // 这块不理解??
            if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }

        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        subscribedEvents.add(eventType);

        // 这个方法是否是sticky方法
        if (subscriberMethod.sticky) {
            if (eventInheritance) {
                // Existing sticky events of all subclasses of eventType have to be considered.
                // Note: Iterating over all events may be inefficient with lots of sticky events,
                // thus data structure should be changed to allow a more efficient lookup
                // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
                Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
                for (Map.Entry<Class<?>, Object> entry : entries) {
                    Class<?> candidateEventType = entry.getKey();
                    if (eventType.isAssignableFrom(candidateEventType)) {
                        Object stickyEvent = entry.getValue();
                        checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                    }
                }
            } else {
                Object stickyEvent = stickyEvents.get(eventType);
                checkPostStickyEventToSubscription(newSubscription, stickyEvent);
            }
        }
    }
```

上面的方法就是按有`@Subscriber`注解的方法的参数类型作为key(即开发声明的MessageEvent),将订阅的class和method封装为Subscripution的list作为value,存储在`subscriptionsByEventType`中.
还将订阅的class作为可以,有`@Subscriber`注解的方法的参数类型作为value,存储在`typesBySubscriber`.
sticky先不分析,至此,`register`方法的流程已经梳理完毕.


#### post方法

接下来我们得从另外一个类中调用`EventBus.getDefault().post(AnyEvent)`方法来通知订阅者们(即注册了事件并且有Subscriber注解的方法).我们来看看这个`post`方法.

```java
 public void post(Object event) {
        // currentPostingThreadState是个ThreadLocal它保证在一个线程中有唯一的值.
        PostingThreadState postingState = currentPostingThreadState.get();
        List<Object> eventQueue = postingState.eventQueue;
        // 把要通知的事件加入到事件队列中
        eventQueue.add(event);
        // isPosting是默认为false的
        if (!postingState.isPosting) {
            postingState.isMainThread = isMainThread();
            postingState.isPosting = true;
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
            }
            try {
                // 这里是依次从事件队列中拿出第一个元素来分发消息直到事件队列为空为止.
                while (!eventQueue.isEmpty()) {
                    postSingleEvent(eventQueue.remove(0), postingState);
                }
            } finally {
                postingState.isPosting = false;
                postingState.isMainThread = false;
            }
        }
    }
```
这里主要步骤就是将要推送的事件加入到队列中,然后循环队列每次获取队列中的第一个元素来进行事件推送,直到事件队列为空为止.接下来我们看看`postSingleEvent`这个函数是干了什么?

```java
 private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
        Class<?> eventClass = event.getClass();
        boolean subscriptionFound = false;
        if (eventInheritance) {
            // 查询具有该通知事件的所有类
            List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
            int countTypes = eventTypes.size();
            for (int h = 0; h < countTypes; h++) {
                Class<?> clazz = eventTypes.get(h);
                // 通知每个类(具有该通知消息的类)
                subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
            }
        } else {
            subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
        }
        if (!subscriptionFound) {
            if (logNoSubscriberMessages) {
                logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
            }
            if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                    eventClass != SubscriberExceptionEvent.class) {
                post(new NoSubscriberEvent(this, event));
            }
        }
    }
```
我们看到第一步是通过调用`lookupAllEventTypes`方法,获取所有注册该消息的类,然后遍历所有的类依次调用`postSingleEventForEventType`方法.我们看看`postSingleEventForEventType`的实现.

```java
  private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
        CopyOnWriteArrayList<Subscription> subscriptions;
        synchronized (this) {
            subscriptions = subscriptionsByEventType.get(eventClass);
        }
        if (subscriptions != null && !subscriptions.isEmpty()) {
            for (Subscription subscription : subscriptions) {
                postingState.event = event;
                postingState.subscription = subscription;
                boolean aborted = false;
                try {
                    postToSubscription(subscription, event, postingState.isMainThread);
                    aborted = postingState.canceled;
                } finally {
                    postingState.event = null;
                    postingState.subscription = null;
                    postingState.canceled = false;
                }
                if (aborted) {
                    break;
                }
            }
            return true;
        }
        return false;
    }
```

```java
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
            case POSTING:
                invokeSubscriber(subscription, event);
                break;
            case MAIN:
                if (isMainThread) {
                    invokeSubscriber(subscription, event);
                } else {
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
            case MAIN_ORDERED:
                if (mainThreadPoster != null) {
                    mainThreadPoster.enqueue(subscription, event);
                } else {
                    // temporary: technically not correct as poster not decoupled from subscriber
                    invokeSubscriber(subscription, event);
                }
                break;
            case BACKGROUND:
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    invokeSubscriber(subscription, event);
                }
                break;
            case ASYNC:
                asyncPoster.enqueue(subscription, event);
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }
```

```java
void invokeSubscriber(Subscription subscription, Object event) {
        try {
            subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
        } catch (InvocationTargetException e) {
            handleSubscriberException(subscription, event, e.getCause());
        } catch (IllegalAccessException e) {
            throw new IllegalStateException("Unexpected exception", e);
        }
    }
```

通过反射调用具有`Subscriber`注解的方法,至此分析完毕.