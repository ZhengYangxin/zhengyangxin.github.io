---
title: OkHttp
copyright: true
date: 2019-05-20 23:02:01
tags: 网络，Okhttp
category: Android
---

#### 一. 曾经的网络框架
##### Android的网络框架有四种
1. HttpClient ： 2.2版本以下存在bug，所以2.3之后建议使用HttpUrlConnection
2. HttpUrlConnection 2.3+， 官方推荐
2. Volley，谷歌开发，简单的网络任务框架，底层兼容2.3以前版本使用了HttpClient，2.3+使用的是HttpUrlConnection，功能拓展性弱
3. Okhttp，从Android4.4开始HttpURLConnection的底层实现采用的是okHttp.

##### 优缺点
![](/image/compare.png)
通过上面的比较，在Android发展的每个阶段，他们有各自存在的意义，只是时过境迁，被种种原因被替换或者废弃。
这个过程是一个框架发展的过程，从重量级繁杂且难以维护，到轻量级易扩展，到过度版本新老版本的兼容，最后取长补短完善自己的网络框架。
化繁为简，然后又能包容万象的过程

##### 网络框架应该有的功能
1. 自定义请求的Header
2. GET，POST
3. 支持文件上传下载
4. 图片加载
5. 支持多任务网络请求操作
6. 支持缓存
7. 支持回调
8. 支持session
9. .....

#### 二. OkHttp简介
一张图了解OkHttp的整个过程
![](/images/OkHttp.jpg)
在OkHttp中真正核心的东西是Interceptor，他不仅负责拦截请求进行额外的处理(入cookie)，实际上他还会把实际的网络请求，缓存，透明压缩等功能都统一起来，每一个功能都只是一个Interceptor，它们在连接成一个Interceptor.Chain,环环相扣最终完成一次网络请求，从getResponseWithInterceptorChain函数中我们可以看到Interceptor.Chain的分布情况依次是：
1. 在配置OkHttpClinet时设置的interceptors
2. 负责失败重试和重定向的RetryAndFollowUpInterceptor
3. 负责把用户构造的请求转化为发送到服务器的请求，把服务器返回的响应转化为用户友好的响应BridgeInterceptor
4. 负责读取缓存直接返回，更新缓存的CacheInterceptor
5. 负责和服务器建立连接的ConnectInterceptor
6. 配置OkHttpClient时设置的NetworkInterceptor
7. 负责向服务器发送请求数据，从服务器读取响应数据的CallServerInterceptor
 
在这里位置决定了功能，最后一个一定是CallServerInterceptor，其他的在这之前。责任链模式在Interceptor中得到了很好的实践。对于request变成response对象，每个interceptor都能完成这件事，也由各自的inteceptor决定是否要交给下个interceptor。

#### 三. Interceptor分析
首先看分析ConnectInterceptor和CallServerInterceptor，这两个interceptor实现了和服务器进行通信的核心
##### 1. ConnectInterceptor建立连接
```
/** Opens a connection to the target server and proceeds to the next interceptor. */

  @Override 
  public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Request request = realChain.request();
    Transmitter transmitter = realChain.transmitter();

    // We need the network to satisfy this request. Possibly for validating a conditional GET.
    boolean doExtensiveHealthChecks = !request.method().equals("GET");
    Exchange exchange = transmitter.newExchange(chain, doExtensiveHealthChecks);

    return realChain.proceed(request, transmitter, exchange);
  }
```
通过创建一个Exchange对象，他将在后面使用。他的内部是对http，https请求的实现，内部都是利用Okio对Socket的读写操作进行了封装.在内部是对java.io和java.nio进行了封装，内部创建了一个主要的RealConnectionn对象，利用RealConnectionn进行读写

##### 2. CallServerInterceptor 发送和接受数据
主要过程：
1. 向服务器发送request header
2. 如果有request body,就向服务器发送
3. 读取response header, 构造response 对象
4. 如果有response body,则创建一个带body的response对象

##### 3. CacheIntercepter 缓存
在建立连接，和服务器通讯之前就是CacheIntercepter，我们需要检查响应是否已经本地缓存了，如果缓存了则直接返回，否则进行后面的流程，并把返回的数据写入缓存
1. 获取本地缓存cacheCandidate
2. 如果本地缓存可用则直接返回CacheCandidate，从而打断interceptor链
3. 走剩下的interceptor获取nnetworkResponse
4. networkResponse、cacheResponse构造新的response
5. 根据新的response里的header定制缓存策略，存入缓存中（method 为get）


#### 总结
创建一个单例的OkHttpClient，创建请求对象request，初始化请求方式，请求url，请求header，请求body，然后通过client的newcall（request）构建真正的请求对象realcall。有realcall的execute方法和qnqueue方法区分是同步请求还是异步请求，异步请求依赖线程池dispatcher，最终会调用getResponseWithInterceptorsChain方法返回返回response。内部通过这种拦截器对request请求数据和response响应数据进行处理，每个拦截器直接通过realinterceptchain对象的process连接起来（责任链模式）
