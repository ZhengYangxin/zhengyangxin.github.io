---
title: OkHttp的学习
copyright: true
date: 2019-10-29 21:34:11
tags: [OkHttp]
category: Android
---
### 学习大纲
1. OSI七层模型介绍，TCP/IP模型介绍，Http协议的格式介绍
2. OkHttp主线流程的源码阅读
3. OkHttp源码阅读之线程池详解
4. OkHttp责任链模式/建造者模式
5. OkHttp整体框架
6. OkHttp之Socket的请求与实现

### 1. 网路模型
1. OSI七层模型，数据封装，解封装的过程
2. TCP/IP模型， 应用层，传输层，网络层， 主机到网络层
3. Http1.0, 请求响应后会马上断开；Http1.1,添加了keepAlive的长连接保持
4. get请求，请求行，请求属性集；post请求，请求行，请求属性集，请求体长度，请求体类型

### 2. OkHttp主流程源码阅读
#### 使用
    1. 创建OkHttpClient的实例对象client
    2. 创建请求对象Request的实例对象request
    3. 通过client.newCall(request)获取到请求对象call
    4. 通过请求对象call进行同步call.execute()请求或者call.enqueue(new CallBack())的异步请求
#### 主流程
    1. 通过构建者设计模式，创建出OkHttpClient的实例
    2. 同样通过构建者设计模式，创建出Request的实例
    3. client.newCall(request)内部通过Call的实现类RealCall，返回call的对象
    4. 进行异步请求call.enqueue(new CallBack())不能重复请求，会调用分发器dispatcher.enqueue(new AsyncCall(responseCallback))
    5. 在dispatcher中定义了双端的runningAsyncCalls和readyAsyncCalls及限制了同时访问同一个服务器为最大5个
    6. 通过线程池调用executor.execute(asyncCall), asyncCall实现了Runnable
    7. AsyncCall内部执行耗时execute方法，会调用责任链获得响应response
    8. 成功与失败的回调，并且进行错误责任的划分

### 3. OkHttp源码阅读之线程池详解
```
public synchronized ExecutorService executorService() {
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }
```
1. OkHttp中使用的线程池，采用了缓存方案。线程任务60s内执行完毕，则会复用线程。自定义线程工厂
2. 守护线程的设置

### 4. OkHttp责任链模式/建造者模式
责任链模式关键是，拦截器集合，拦截器的一个管理类，index+1 
建造者模式关键是构建对象

### OkHttp整体框架
1. http的默认端口是80， https是443
2. callServerIntercept 建立连接，会做连接池的复用
3. 网络层是在路由器之间的传输ip数据包，数据链路层是交换机间的传输是帧，最后是物理机的比特流的传输
4. OkHttp封装了Socket,对socket进行了复用，缓存机制
5. 连接池的设计，定义了一个线程池和一个清理连接对象的线程，通过连接对象的空闲时间和允许的最大空闲时间来回收连接池

