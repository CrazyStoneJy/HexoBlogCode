---
title: okhttp源码学习
date: 2017-11-29 16:11:52
tags: [okhttp,net]
categories: [android]
---

大家都已经对okhttp很熟悉了吧,今天我把我学习okhttp源码的过程记录一下.
okhttp是一个轻量级的网络请求框架,可用于java和android,支持同步请求和异步请求两种方式.

## **同步GET请求源码分析**

```java
    //实例okhttpclient
    OkhttpClient client = new OkhttpClient.Builder().build();
    //实例Request
    Request request = new Request.Builder().url('http://www.baidu.com').build();
    //发送请求
    new Thread(
        public void run(){
            Response response = client.newCall(request).execute();
        }
    ).start();
```
以上是简单的OkHttp同步GET请求方式,接下来咱们来分析一下源码.

<!-- more -->

### 1.实例化OkHttpClient

首先需要获取一个OkHttpClient的实例,可以通过直接new,也可通过new OkHttpClient内部的静态内部类Builder,通过调用build方法获取.直接new OkHttpClient对象,其实也是通过new OkHttpClient.Builder()
```java
 public OkHttpClient() {
    this(new Builder());
  }
```
> OkHttp.Builder属于建造者模式,这样写的好处可以使代码写的更优雅一些,链式配置属性是不是很舒服.

可以看到在new OkHttpClient.Builder()时,创建了Dispatcher对象和ConnectionPool对象,Dispatcher主要是用来网络分发的(用来处理同步请求线程和异步请求线程),ConnectionPool是用来http请求的连接池.

```java
public Builder() {
      //创建网络分发器
      dispatcher = new Dispatcher();
      protocols = DEFAULT_PROTOCOLS;
      connectionSpecs = DEFAULT_CONNECTION_SPECS;
      eventListenerFactory = EventListener.factory(EventListener.NONE);
      proxySelector = ProxySelector.getDefault();
      cookieJar = CookieJar.NO_COOKIES;
      socketFactory = SocketFactory.getDefault();
      hostnameVerifier = OkHostnameVerifier.INSTANCE;
      certificatePinner = CertificatePinner.DEFAULT;
      proxyAuthenticator = Authenticator.NONE;
      authenticator = Authenticator.NONE;
      //创建http连接池
      connectionPool = new ConnectionPool();
      dns = Dns.SYSTEM;
      followSslRedirects = true;
      followRedirects = true;
      retryOnConnectionFailure = true;
      connectTimeout = 10_000;
      readTimeout = 10_000;
      writeTimeout = 10_000;
      pingInterval = 0;
    }
```

在Dispatcher中维护着一个用来管理异步请求线程的线程池,ConnectionPool中也维护着一个连接http请求的线程池,这也是为什么我们在网络请求中只用一个OkHttpClient的原因,或者用单例封装起来.如果创建多个OkHttpClient对象,那么DisPatcher和ConnectionPool的线程池也会多次创建,这样会消耗CPU资源和内存资源.

这是Dispatcher的线程池
```java
    public synchronized ExecutorService executorService() {
        if (executorService == null) {
            executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp Dispatcher", false));
        }
        return executorService;
    }
```
ConnectionPool的线程池
```java
    private static final Executor executor = new ThreadPoolExecutor(0 /* corePoolSize */,Integer.MAX_VALUE /* maximumPoolSize */, 60L /* keepAliveTime */, TimeUnit.SECONDS,new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp ConnectionPool", true));
```

### 2.创建Request,调用OkHttpClient的newCall方法

根据不同的http请求方式创建不同的Request对象,这个我们先不研究,后面再说.先看创建完Request对象后,调用client.newCall(request)的源码.

```java
  /**
   * Prepares the {@code request} to be executed at some point in the future.
   */
  @Override public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
  }
```
我们看到client.newCall(request)方法实现中,其实是调用了RealCall对象的newRealCall的静态方法,那么我们接下来看看newRealCall方法里面是进行了什么操作.

```java
static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    // Safely publish the Call instance to the EventListener.
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    call.eventListener = client.eventListenerFactory().create(call);
    return call;
  }
```
RealCall中的newRealCall方法中实际上就是new了一个RealCall实例,并且创建了一个网络请求监听事件的Listener.接下来,我们将要分析的是同步请求,RealCall的execute方法.

### 3.同步请求execute方法

```java
 @Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    try {
        //执行同步请求
      client.dispatcher().executed(this);
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } catch (IOException e) {
      eventListener.callFailed(this, e);
      throw e;
    } finally {
      client.dispatcher().finished(this);
    }
  }
```

可见execute方法最后返回一个Response对象.咱们捡关键的先看:client.dispatcher().executed(this);Response result = getResponseWithInterceptorChain();这两个方法是实现的关键部分.先分析第一个方法,client.dispatcher().executed(this)是调用了Dispatcher对象中的executed方法,那我们再看看它的实现.

```java
/** Used by {@code Call#execute} to signal it is in-flight. */
  synchronized void executed(RealCall call) {
    runningSyncCalls.add(call);
  }
```
可见executed方法只是将该请求加入到了一个同步的队列中.

```java
 /** Ready async calls in the order they'll be run. */
  //异步请求等待队列
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  /** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
  //异步请求执行队列
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  /** Running synchronous calls. Includes canceled calls that haven't finished yet. */
  //同步请求执行队列
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
```
这是Dispatcher中的三个请求队列:runningSyncCalls是个同步的请求队列,runningAsyncCalls是在运行当中的异步请求队列,readyAsyncCalls是在等待中的异步请求队列.(异步请求是有最大请求数的,后面说)

接下来我们看看OkHttp中网络请求核心部分getResponseWithInterceptorChain().
```java
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    //自定义拦截器
    interceptors.addAll(client.interceptors());
    //失败重连拦截器
    interceptors.add(retryAndFollowUpInterceptor);
    //处理请求头拦截器(header)
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    //缓存拦截器
    interceptors.add(new CacheInterceptor(client.internalCache()));
    //http连接拦截器
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      //web socket拦截器
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    return chain.proceed(originalRequest);
  }
```
可见该方法是创建了一个拦截器队列,使用了责任链模式来一层一层的对Request和Response进行操作,最终返回一个Response对象.
这里是OkHttp责任链模式的简写方式:[责任链模式]('https://github.com/CrazyStoneJy/DesignPattern/tree/master/src/study/crazystone/me/responsibility_chain')

以上就是OkHttp同步网络请求的大致流程.

## OkHttp异步网络请求

异步请求和同步请求的最大区别是:同步请求需要自己创建线程管理线程,而异步请求则是用OkHttp Dispatcher中的线程进行请求.client.enqueue(request)实际上是调用的RealCall的enqueue方法,我们看看RealCall的enqueue.

```java
@Override public void enqueue(Callback responseCallback) {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
  }
```
可见RealCall的enqueue方法实际调用的是Dispatcher的enqueue方法,我们继续看Dispatcher的enqueue方法.

```java
  synchronized void enqueue(AsyncCall call) {
    //运行中的异步请求队列大小小于最大值(默认值为64,可以更改)并且异步请求队列中url的host个数小于最大值(host的最大值默认为5,也可以更改)
    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      runningAsyncCalls.add(call);
      executorService().execute(call);
    } else {
      readyAsyncCalls.add(call);
    }
  }
```
如果**运行异步队列**小于maxRequests and 异步请求的host的个数小于maxRequestsPerHost,则将该次请求加入到**运行异步队列**然后运行该线程;否则,将该请求队列加入到**待请求的异步队列**.AysncCall这是一个定制的线程,接下来看看该线程的run方法怎么执行的.

```java
final class AsyncCall extends NamedRunnable
```
看到AsynCall实际继承的是NamedRunnable,NamedRunnable是个抽象类,我们再看看NamedRunnable的run方法

```java
@Override public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }

  protected abstract void execute();
```
看到execute这个方法是个钩子方法,execute方法在run方法中被调用,execute是个抽象方法,子类必须实现该方法,因此,我们看看AysnCall的execute方法.

```java
@Override protected void execute() {
      boolean signalledCallback = false;
      try {
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          eventListener.callFailed(RealCall.this, e);
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        client.dispatcher().finished(this);
      }
    }
```
看到AysnCall中execute方法,调用了与同步请求的相同的getResponseWithInterceptorChain的方法,getResponseWithInterceptorChain是具体处理网络请求返回Response的方法.在获取到reseponse后,finally中又调用了Dispatcher的finished方法.

```java
 void finished(AsyncCall call) {
    finished(runningAsyncCalls, call, true);
  }

 private <T> void finished(Deque<T> calls, T call, boolean promoteCalls) {
    int runningCallsCount;
    Runnable idleCallback;
    synchronized (this) {
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      if (promoteCalls) promoteCalls();
      runningCallsCount = runningCallsCount();
      idleCallback = this.idleCallback;
    }

    if (runningCallsCount == 0 && idleCallback != null) {
      idleCallback.run();
    }
  }
```
finished方法主要是将执行完后的线程从**运行请求队列**中移除,promoteCalls为true表示执行下一个请求,promoteCalls方法被调用.

```java
 private void promoteCalls() {
    //如果运行中的线程队列大小大于最大请求数则直接return
    if (runningAsyncCalls.size() >= maxRequests) return; // Already running max capacity.
    //如果待请求队列为空则直接return
    if (readyAsyncCalls.isEmpty()) return; // No ready calls to promote.
    //遍历待请求队列如果运行中队列的host个数小于maxRequestsPerHost
    //便将该线程从待请求队列中移除,加入到运行中队列,并执行该线程.
    for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
      AsyncCall call = i.next();

      if (runningCallsForHost(call) < maxRequestsPerHost) {
        i.remove();
        runningAsyncCalls.add(call);
        executorService().execute(call);
      }

      if (runningAsyncCalls.size() >= maxRequests) return; // Reached max capacity.
    }
  }
```
promoteCalls方法是将**待请求队列**中的线程移动到**运行中队列**.

## OKHttp Intercept

这里是对okhttp拦截器的总结
|拦截器|作用|
|----|----|
|client.interceptors()|开发者自定义的拦截器|
|retryAndFollowUpInterceptor|失败重定向拦截器(内部实现还未研究明白)|
|BridgeInterceptor|主要用作生成或者解析http的报文格式|
|CacheInterceptor|响应体的缓存|
|ConnectInterceptor|连接拦截器|
|client.networkInterceptors()|开发者自定的网络拦截器|
|CallServerInterceptor|请求服务器的拦截器|




