---
title: 跟我学：Android高级面试之说说Android系统的启动流程
copyright: true
date: 2020-04-03 06:31:29
tags:
category:
---

### 导读

本人从事Android客户端开发，最近在在深入Framwork学习，希望我的学习分享能帮到你。首先我们看一下关于Android系统的启动流程我们应该如何去分析，这里通过一张思维导图帮助大家整理思路。思维导图一方面罗列了本文的大纲，另一方面也是希望读者朋友可以保存图片至自己的复习资料里。以便随时通过该大纲去复习，可以关注我，收藏文章进一步沟通学习！

![跟我学：Android高级面试之说说Android系统的启动流程](http://p1.pstatp.com/large/pgc-image/09c554f2e27f45639de7b22005dc7389)

### 说说Android系统的启动流程

面对这道面试题，面试官试想考察什么呢？

- Android中有哪些主要的系统进程
- 这些系统进程是怎么启动的
- 进程启动之后主要做了什么事

### 系统进程

Android中有哪些系统进程，具体我们可以在init.rc的配置找到相应的启动代码

```
start zygote
start servicemanager
start surfaceflinger
....
```

上面几个熟悉的服务进程都是通过init单独创建启动的，还有 一个我们熟悉的SystemServer，这个服务进程是通过Zygote进程创建而不是init进程。

### Zygote的启动流程

关于Zygote的的相关知识可以参考[跟我学：Android面试之深入Android Framework谈谈对Zygot的理解](https://www.toutiao.com/i6810136009775251975/?group_id=6810136009775251975)

它的启动流程是

1. init进程fork出zygote进程
2. zygote创建虚拟机，注册jni函数
3. 预加载系统资源
4. 启动SystemServer
5. 进入 Socket LOOP循环（工作原理）

### SystemServer的启动

在ZygoteInit.java的代码逻辑中，可以看到启动的关键代码

```
int pid = Zygote.forkSystemServer(...) // 通过Zygote创建子线程
handleSystemServerProcess(...)  // 然后初始化SystemServer
ZygoteInit.zygoteInit(...)
ZygoteInit.nativeZygoteInit(...); // 这里面会启动启动binder机制，创建了Binder线程
RuntimeInit.applicationInit(...) // 调用SystemServer.java类的入口函数main。内部基于反射实现
       
// SystemServer的main方法
public static void main(String[] args) {
        new SystemServer().run();
}

run方法中主要以下几件事
1. Looper.prepareMainLooper(); // 为主线程创建loop
2. System.loadLibrary("android_servers"); // 加载android的共享库，native层的代码
3. createSystemContext(); // 创建系统上下文
4. 分批启动服务startBootstrapServices();startCoreServices();startOtherServices();
5. Looper.loop(); // 进入loop循环，主线程不会退出
```

### 系统服务的启动

系统服务的启动指startBootstrapServices();startCoreServices();startOtherServices();内部的服务启动。启动startBootstrapServices()中包括了熟悉的AMS，PMS，StartPowerManager等等。startCoreServices()包含了BatteryService，UsageStatsService，WebViewUpdateService等等。以及startOtherServices()中的其他服务。

#### 系统服务如何房补，让应用程序可见

通过源码的阅读，我们可以发现最终服务是通过ServiceManager.addService(...) 将服务进行注册，然后才可以可被其他进程可见和使用

#### 系统服务运行在什么线程中？

主线程：并没有找到哪个系统服务是在主线程的中的

工作线程： AMS，PMS运行在自己的工作线程，或者在公用的工作线程DisplayThread，FgThread, IoThread, UiThread。

Binder线程：应用在跨进程调用时肯定是在binder线程里的，然后再去切换线程

### 如何解决系统服务启动的相互依赖

从上小结的startBootstrapServices();startCoreServices();startOtherServices();我们可以看出解决方式为

- 分批启动：先启动AMS，PMS等，因为很多其他service都需要依赖他们
- 分阶段启动：在每个阶段去启动相应需要的服务

### 桌面启动

当AMS等其他服务都启动完毕，会调用mActivityManagerService.systemReady(()，在这里面会通过startHomeActivityLocked(currentUserId, "systemReady");启动桌面应用的activity， Launcher.java。启动后会通过PMS查找所有的已安装的应用，然后显示在界面上

### 最后

看到最后，我相信你一定对Android系统的启动流程有一定了解了吧。回头看看第一部分的考查内容讲讲呗，对于每一个点的重点我都细心的为你做了高亮。还有纸上得来终觉浅，绝知此事要躬行。本文参考了Android api28的源码，希望你也去踩着点过一遍源码，肯定能加深理解！

我是Yangcy，该吃吃该喝喝，该学还得学，我们一起加油！