---
title: Retrofit源码解析
date: 2018-03-07 15:23:18
tags: [android,Retrofit,源码解析]
categories: [android,源码解析]
---

Retrofit这个框架字如其面:重构,即对OKHttp的重构.是使用注解+动态代理的方式是开发者的开发工作变的更加方便,其次是为了开发者在调用RESTful风格的后台接口更加方便.

我们这里先回忆一下Retrofit的使用,首先我们需要定一个接口(这个接口不能在继承其他接口了),为什么一会分析源码是讲解.好,我现在来编写这个接口.

```java
public interface Api(){
    // 这里是url的相对路径
    @GET('/v1/{p}/test')
    Call<String> test(@Path("p")String person,@Query String a);
}
```

<!--  more -->

恩,很好,我们定义好了一个Retrofit想要的接口的形式,接下来我们开始写Retrofit的设置及调用代码.

```java
public NetHelper {
    public static Api get(){
        Retrofit retrofit = Retrofit.Builder()
            // 设置你的host的url
            .baseUrl("http://www.baidu.com/")
            // 配置okhttp
            .client(new OkHttpClient())
            // 设置解析数据的工厂
            .addConverterFactory(GsonConverterFactory.create())
            .build();
        Api api = retrofit.create(Api.class);
        Call<String> call = api.test("lisi","test");
        call.enqueue(new Callback<String>() {
            @Override
            public void onResponse(Call<String> call, Response<String> response) {
                
            }

            @Override
            public void onFailure(Call<String> call, Throwable t) {

            }
        });
    }
}
```

好,以上就是使用Retrofit完成了一个异步网络请求.接下来我们分析分析Retrofit的源码实现.

### Builder

首先是Retrofit的静态内部类Builder,是用构造者的模式配置了一系列的东西.主要看几个方法:`baseUrl`,`addConverterFactory`,`addConverterFactory`,`client`.

1. 设置host的url

```java
public Builder baseUrl(String baseUrl) {
      checkNotNull(baseUrl, "baseUrl == null");
      HttpUrl httpUrl = HttpUrl.parse(baseUrl);
      if (httpUrl == null) {
        throw new IllegalArgumentException("Illegal URL: " + baseUrl);
      }
      return baseUrl(httpUrl);
    }
```

2. 配置response解析工厂

```java
 public Builder addConverterFactory(Converter.Factory factory) {
      converterFactories.add(checkNotNull(factory, "factory == null"));
      return this;
    }
```
可以看到这是将新的解析工厂加入到一个list中,当发生网络请求获取到Response之后,会遍历解析list的挨个取解析,因此可以添加多个解析工厂.

3. 配置返回的Call的类型的工厂

```java
public Builder addCallAdapterFactory(CallAdapter.Factory factory) {
      adapterFactories.add(checkNotNull(factory, "factory == null"));
      return this;
    }
```

4. 配置OKHttpClient

```java
public Builder client(OkHttpClient client) {
      return callFactory(checkNotNull(client, "client == null"));
    }
```

### create

接下来我们分析这个Retrofit库中非常重要的一个方法`create`,这个方法内部实现是用了动态代理.动态代理的好处是什么?那我们先想一想如果没有动态代理,只使用静态代理,如果有许许多多的类需要写代理类,那重复代码和工作量是不是很大,动态代理就是要来简化工作量的.怎么简化工作量?因为,动态代理实际上是使用的反射,通过反射获取要代理的类的内部方法,使用`method.invoke(obj,args);`的方式来调用它,因此只要传递进来类名我们便可以调用它的方法实现,不用再写多个代理类,从而简化了工作量.好,我们分析完了动态代理原理,我们看看`create`的实现.

```java
 public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    // 动态代理,使用Proxy的一个静态方法newProxyInstance,需要实现InvocationHandler接口
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            // 如果是method是Object的方法,直接正常调用
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
  }
```
我们看到`InvaocationHandler`回调方法中,有个`loadServiceMethod`方法,这是个很重要的方法,我在看看它的实现.

```java
 ServiceMethod<?, ?> loadServiceMethod(Method method) {
    // double lock check 
    ServiceMethod<?, ?> result = serviceMethodCache.get(method);
    if (result != null) return result;

    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        result = new ServiceMethod.Builder<>(this, method).build();
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }
```

<!-- 这个方法的意思是:先从`serviceMethodCache`缓存中取,如果通过key为该method获取到了值,则直接返回,否则执行一个同步代码块,将 -->


