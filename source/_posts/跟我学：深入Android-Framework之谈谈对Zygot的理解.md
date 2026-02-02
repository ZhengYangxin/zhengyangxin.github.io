---
title: 跟我学：深入Android Framework之谈谈对Zygote的理解
copyright: true
date: 2020-03-31 12:59:49
tags: [Zygote]
category: Android
author: Yangcy
summary: 谈谈对Zygote的理解，深入分析从Zygote的作用，启动流程到工作原理
---

最近在在深入Framwork学习，希望我的学习分享能帮到你。首先我们看一下关于Zygote我们应该如何去分析，这里通过一张思维导图帮助大家整理思路。思维导图一方面罗列了本文的大纲，另一方面也是希望读者朋友可以保存图片至自己的复习资料里。以便随时通过该大纲去复习！

<img src="/images/image-F8C39DF903A8-1.png" style="zoom:14%;" />

### 谈谈对Zygote的理解

当遇到这样一道面试题，我们应该分析面试官想考察的是什么？

- 了解Zygote的作用，初级的要求，答出来后方能深入
- 熟悉Zygote的启动流程，中级要求，主要是启动中做的事有哪些关键步骤
- 深刻理解Zygote的工作原理，高级要求，主要是怎么启动进程的，怎么与其他进程通讯

### Zygote的作用

他的作用非常简单就两点

- 启动SystemServer
- 孵化应用进程

如果答出了这两点那就是及格了。但可能大部分朋友只知道第二点，第一点就不是那么清楚。其实SystemServer也是Zygote启动的，因为SystemServer需要用到Zygote准备好的系统资源：包括我们常用的一些类、注册的JNI函数、主题资源及一些共享库等等，直接从Zygote继承过来自己就不需要重新加载过来，那么对性能将会有很大的提升。

### Zygote的启动流程

在说Zygote启动流程之前，我们可以明确一个概念：**启动三段式**，这个可以理解为Android中进程启动的常用套路，分为三步骤

1. 进程启动
2. 准备工作
3. LOOP循环

其中LOOP作用是**不停的接受消息，处理消息**，消息的来源可以是Soket、MessageQueue、Binder驱动发过来的消息，但无论消息从哪里来，它总的流程都是去接受消息，处理消息。

这个启动三段式，他不光是Zygote进程是这样的，只要是有独立进程的，比如说系统服务进程，自己的应用进程都是这样的。

#### Zygote进程是怎么启动的？

Zygote进程的启动，得感谢init进程。init进程是它是linux启动之后用户空间的第一个进程。下面看一下**启动流程**

1. linux启动init进程
2. 加载init.rc配置文件
3. 启动配置文件中定义的系统服务，其中Zygote服务就是定义在配置中的
4. init进程通过fork + execve 系统调用启动Zygote

在分析具体系统源码实现之前，我分享一个看源码很方便网站Android社区：www.androidos.net.cn

下面列举了本文需要阅读的源码文件路径，供大家方便查找。

```
platform/system/core/rootdir/init.zygoteXX.rc
platform/system/core/rootdir/init.rc
platform/frameworks/base/cmds/app_process/app_main.cpp
platform/frameworks/base/core/jni/AndroidRuntime.cpp
platform/libnativehelper/JniInvocation.cpp
platform/frameworks/base/core/java/com/android/internal/os/ZygoteInit.java
```

#### 加载Zygot的启动配置

在init.rc 文件中会import /init.${ro.zygote}.rc，init.zygoteXX,XX指的是32或者64，对我们没差我们直接看init.zygote32.rc即可。配置文件比较长，我这里做了截取保留了Zygot相关的部分。

```
service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
    class main
    socket zygote stream 660 root system
    onrestart write /sys/android_power/request_state wake
    onrestart write /sys/power/state on
    onrestart restart audioserver
    writepid /dev/cpuset/foreground/tasks
```

- service zygote：是进程名称， 
- /system/bin/app_process：可执行程序的路径，用于init进程fork，execve调用
- -Xzygote /system/bin --zygote --start-system-server 为它的参数

#### 启动方式

获取到启动配置后才会进行正式启动，启动方式有两种

- fork + handle 
- fork + execve

首先都会调用fork创建新的进程，比较奇特的是它会返回两次。

- 子进程一次，返回的pid是0 父进程一次，返回的pid是子进程的pid
- 对于handle默认的情况，子进程会继承父进程的所有资源，但当通过execve去加载二进制程序时，那父进程的资源则会被清除

#### 信号处理-SIGCHLD。

当父进程fork子进程后，父进程需要关注这个信号。当子进程挂了，父进程就会收到SIGCHLD，这时候父进程就可以做一些处理。例如Zygote进程如果挂了，那父进程init进程就会收到信号将Zygote进程重启

#### Zygote进程启动后做了什么

主要分为两部分Native层处理和Java层处理。

先来看一下Native层的处理

- 启动Android虚拟机
- 注册Android的JNI函数
- 进入Java层

在app_main.cpp文件，AndroidRuntime.cpp文件。我们可以找到几个主要函数名

```
JNI_CreateJavaVM   // 创建启动虚拟机
jniRegisterNativeMethods(env, "com/android/internal/os/ZygoteInit",
        methods, NELEM(methods)) // 通过反射获取ZygoteInit对象
env->CallStaticVoidMethod(startClass, startMeth, strArray);//调用main函数,进入Java层
```

在应用进程中并不需要创建虚拟机，因为应用进程是Zygote进程孵化出来的，继承了父进程的拥有虚拟机，只需要重置数据即可。

接着看一下Java层的处理，具体可参考ZygoteInit文件的main方法

- 预加载资源，常用类库、主题资源及一些共享库等
- 启动SystemServer进程
- 进入Socket 的Loop循环 会看到的ZygoteServer.runSelectLoop(...)调用



### Zygote的工作原理

Zygote的LOOP循环会一直监听Socket中的内容，所以Zygote与其他进程的通信是通过Socket的，工作原理举个例子

1. 桌面应用点击某应用图标，若应用没有启动进程
2. AMS会通过Socket通知Zygote进程
3. Zygote进程接受到消息后fork出一个应用进程
4. 执行应用进程的ActivityThread的main方法

### 要注意的细节

Zygote fork要单线程进行，在fork时Zygote会将除主进程外的所有线程都停了成功后重启，因为对于新创建的子进程而言只有主线程，避免多线程的死锁问题

Zygote的IPC没有采用binder，而是本地Socket

### 最后

看到最后，我相信你一定对Zygote有一定了解了吧。回头看看第一部分的考查内容讲讲呗，对于每一个点的重点我都细心的为你做了高亮。还有纸上得来终觉浅，绝知此事要躬行。本文参考了Android api28的源码，希望你也去踩着点过一遍源码，肯定能加深理解！

<img src="/images/follow_end_blog.jpg" style="zoom:30%;" />

### 