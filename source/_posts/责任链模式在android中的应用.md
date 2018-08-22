---
title: 责任链模式在android中的应用
date: 2018-06-20 15:50:50
tags: [设计模式]
categories: [设计模式]
---

#### 责任链模式概念

首先先说说什么是责任链模式

> 在责任链模式里，很多对象由每一个对象对其下家的引用而连接起来形成一条链。请求在这个链上传递，直到链上的某一个对象决定处理此请求。发出这个请求的客户端并不知道链上的哪一个对象最终处理这个请求，这使得系统可以在不影响客户端的情况下动态地重新组织和分配责任。

维基百科上对于责任链模式的描述[wikipedia](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)

责任链模式最基本的就是多个分支判断形成的链,怎么说呢,先看看下面的一段伪代码,

```java

private String check(Object condition){

    String result = "";

    if(checkA(condition)){
        return "A符合";
    }

    if(checkB(condition)){
        return "B符合";
    }

    if(checkC(condition)){
        return "c符合"
    }
    return result;
}

```

上面这个代码就是最简单的责任链模式的呈现,但是你可能会说,你tm在逗我,我看到的责任链模式可不张这样.

大兄弟,你别着急,你听我说,你看到的平常的代码中的责任链模式,是将上述代码中的checkA(),checkB()...等抽象成接口,使用面向对象的组合和继承等一系列手法,将代码进行解藕实现的.其实最基本的逻辑就是以上的写法,也是我们在日常中经常写的形式.

为什么要用责任链模式呢?随着逻辑的越来越多,以上的写法会写一大堆if else判断,使得代码非常冗余,而且在更改逻辑时,也非常容易出错.因此,我们可以用责任链模式使代码变的优雅,使代码之间的耦合降低.

责任链模式的UML图:

{% plantuml %}
class Client

class Handler{
    Handler successor;
    # handleRequest();
}

class ConcreteHandlerA{
    # handlerReuqest();
}

class ConcreteHandlerB{
    # handlerReuqest();
}

Client ..> Handler
Handler *-- Handler
ConcreteHandlerA --|> Handler
ConcreteHandlerB --|> Handler
{% endplantuml %}

从上面可以看出职责链包含三个角色：

Handler: 抽象处理者。定义了一个处理请求的方法。所有的处理者都必须实现该抽象类。 
ConcreteHandler: 具体处理者。处理它所负责的请求，同时也可以访问它的后继者。如果它能够处理该请求则处理，否则将请求传递到它的后继者。 
Client: 客户类。


#### 责任链模式在android中的应用

1. **View的事件传递机制.**

以下是一个viewgroup的事件分发的核心伪代码.更多讲解请看**[android view事件分发](https://crazystonejy.coding.me/blog/2018/03/01/android-View%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91/)**

```java

public boolean dispatchTouchEvent(MotionEvent event) {
    boolean consume = false;
    if (onInterceptTouchEvent(event)){
        consume = onTouchEvent(event);
    }else{
        consume = child.dispatchTouchEvent(event);
    }
    return consume;
}

```

以下是android view事件机制的流程图,

![android view 事件机制](http://or5n6ccgu.bkt.clouddn.com/18-6-20/84566146.jpg)

2. **okhttp的拦截器**

我们都知道网络的[OSI模型](https://crazystonejy.coding.me/blog/2018/03/06/OSI%E6%A8%A1%E5%9E%8B/),一个网络通信的形成,都是从应用层进行使用http,https,ftp等应用层的协议进行编码,组织成响应的数据格式;然后在往下传递到传输层组成成报文,数据包的形式;再往下添加ip,传到网络层;在通过一系列的路由转换到达另一台服务器,服务器在一层一层的解码获取响应的数据.

> 网络模块的参考书,推荐:`图解TCP/IP`,`网络是怎样连接的`,`计算机网络自定向下方法`

可见一个网络的数据包操作也是一个链式的,通过每一层对request的处理传递到下一层,获取到response后在一层层的向上传递进行解析response.

接下来咱们分析一下okhttp拦截器部分的源码.

```java

 Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    return chain.proceed(originalRequest);
  }

```

从上述代码可以看到interceptors是将所有的拦截器加到一个list中,然后顺序的调用`RealInterceptorChain`中的`process`来进行相应的处理.

> 对okhttp源码的分析,请看这里,**[传送们](https://crazystonejy.coding.me/blog/2017/11/29/OkHttp%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/)**

如果不理解,我们可以看看,他的接口是怎么定义的.

```java
public interface Interceptor {
  Response intercept(Chain chain) throws IOException;

  interface Chain {
    Request request();

    Response proceed(Request request) throws IOException;

    /**
     * Returns the connection the request will be executed on. This is only available in the chains
     * of network interceptors; for application interceptors this is always null.
     */
    @Nullable Connection connection();

    Call call();

    int connectTimeoutMillis();

    Chain withConnectTimeout(int timeout, TimeUnit unit);

    int readTimeoutMillis();

    Chain withReadTimeout(int timeout, TimeUnit unit);

    int writeTimeoutMillis();

    Chain withWriteTimeout(int timeout, TimeUnit unit);
  }
}
```

拦截器的类图关系:

{% plantuml %}

interface Interceptor
interface Chain
class RetryAndFollowUpInterceptor
class BridgeInterceptor
class CacheInterceptor
class ConnectInterceptor
class CallServerInterceptor
class RealInterceptorChain

RetryAndFollowUpInterceptor ..|> Interceptor 
BridgeInterceptor ..|> Interceptor 
CacheInterceptor ..|> Interceptor 
ConnectInterceptor ..|> Interceptor 
CallServerInterceptor ..|> Interceptor 
RealInterceptorChain ..|> Chain

interface Interceptor {
    Response intercept(Chain chain) throws IOException;
}

interface Chain {
    Response proceed(Request request) throws IOException;
}

{% endplantuml %}



下面是interceptor的调用关系:

{% plantuml %}

RealChain -> CustomInterceptor: request

activate CustomInterceptor

CustomInterceptor -> RetryAndFollowUpInterceptor: request

activate RetryAndFollowUpInterceptor

RetryAndFollowUpInterceptor -> BridgeInterceptor: request

activate BridgeInterceptor

BridgeInterceptor -> CacheInterceptor: request

activate CacheInterceptor

CacheInterceptor -> ConnectInterceptor: request

activate ConnectInterceptor

ConnectInterceptor -> CallServerInterceptor: request

activate CallServerInterceptor

CallServerInterceptor -->  ConnectInterceptor: response

deactivate CallServerInterceptor

ConnectInterceptor --> CacheInterceptor: response

deactivate ConnectInterceptor

CacheInterceptor --> BridgeInterceptor: response

deactivate CacheInterceptor

BridgeInterceptor --> RetryAndFollowUpInterceptor: response

deactivate BridgeInterceptor

RetryAndFollowUpInterceptor --> CustomInterceptor: response

deactivate CustomInterceptor

CustomInterceptor --> RealChain: response

{% endplantuml %}

> 推荐绘制流程图工具:plantUML,[官网](http://plantuml.com/)


#### 责任链模式的demo练习

责任链模式代码练习:
[练习场](https://github.com/CrazyStoneJy/DesignPattern/blob/master/src/study/crazystone/me/chain/package-info.java)

对于okhttp的拦截器的代码简化:
[okhttp拦截器简化](https://github.com/CrazyStoneJy/DesignPattern/blob/master/src/study/crazystone/me/responsibility_chain/package-info.java)

#### 责任链模式在项目中的使用

直接看项目中重构的代码部分


#### 责任链模式与装饰者模式的区别

装饰者模式在java io得到了充分应用,接下来看看装饰者模式的uml图:

{% plantuml %}

Component{
    # operation()
}

ConcreteComponent{
    # operation()
}

Decorator{
    component
    # operation()
}

ConcreteDecorator{
    # operation()
}

ConcreteComponent --|> Component
Decorator --|> Component
Component --* Decorator
ConcreteDecorator --|> Decorator

{% endplantuml %}

`责任链模式`和`装饰者模式`在形式上非常相似，但是实际上是执行上是有不同的．

`责任链模式`是如果你不满足条件则传递给下一个类去执行．
而，`装饰者模式`是执行自身，并且在执行传递的下一个类．

[装饰者模式demo](https://github.com/CrazyStoneJy/DesignPattern/blob/master/src/study/crazystone/me/decorator/package-info.java)