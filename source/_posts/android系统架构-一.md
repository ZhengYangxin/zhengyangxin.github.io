---
title: android系统架构(一)
date: 2017-03-20 18:59:49
tags:
---
### 出发点
&emsp;&emsp;由于在学习Android的时候并没有很系统地进行学习，知识点比较零碎，所以需要将自己的知识点进行系统的整理，那么首要的我需要将android的系统架构搞清楚，这样才能分清楚我所了解的知识点附属于哪个层面，是内核还是应用层，可以进行怎样的扩展，在哪些场景去运用。
### Android架构解析
[Google工程师多图详解Android系统架构](http://mobile.51cto.com/android-235496.htm)
[Android基础之Android系统架构](https://my.oschina.net/fhd/blog/349830)
[Android维基百科](https://zh.wikipedia.org/wiki/Android#Linux.E6.A0.B8.E5.BF.83)
[Android对Linux内核的增强](http://tech.it168.com/a2011/0805/1228/000001228471.shtml)
[Android系统架构详解(2)–Android Runtime](http://www.tuicool.com/articles/EN7nuiN)

以上是我在这个课题下的一些参考博客或者网页链接。里面有对于android架构的一些较好的分析理解，接下来是阅读后自己的一些整理。

Android采用层次化系统架构，官方公布的标准架构如下图所示。

![Alt text](https://upload.wikimedia.org/wikipedia/commons/thumb/a/af/Android-System-Architecture.svg/1000px-Android-System-Architecture.svg.png "Optional title")

Android由底层往上分为4个主要功能层，分别是Linux内核层（Linux Kernel），系统运行时库层（Libraries和Android Runtime），应用程序架构层（Application Framework）和应用程序层（Applications）。

接下来对这几个层面进行逐个分析：

#### Linux内核层

&emsp;&emsp;Android以Linux操作系统内核为基础，借助Linux内核服务实现硬件设备驱动，进程和内存管理，网络协议栈，电源管理，无线通信等核心功能。Android4.0版本之前基于Linux2.6系列内核，4.0及之后的版本使用更新的Linux3.X内核，并且两个开源项目开始有了互通。Linux3.3内核中正式包括一些Android代码，可以直接引导进入Android。Linux3.4增添了电源管理等更多功能，以增加与Android的硬件兼容性，使Android在更多设备上得到支持。直到现在最新的android6.0仍然继续延用着linux3.4.0，而linux最新的版本已经到了4.3系列，那么为什么android没有继续去更新Linuxkernel的版本也是一个值得探讨的课题。 

&emsp;&emsp;Android内核 对Linux内核进行了增强，增加了一些面向移动计算的特有功能。例如，低内存管理器LMK（Low Memory Keller），匿名共享内存（Ashmem）,以及轻量级的进程间通信Binder机制等。这些内核的增强使Android在继承Linux内核安全机制的同时，进一步提升了内存管理，进程间通信等方面的安全性。

#### 硬件抽象层

&emsp;&emsp;内核驱动和用户软件之间还存在所谓的硬件抽象层（Hardware Abstract Layer,HAL），它是对硬件设备的具体实现加以抽象。HAL没有在Android官方系统架构图中标明，下图标出了硬件抽象层在android系统中的位置：

![Alt text](http://static.oschina.net/uploads/space/2014/1128/125552_hnZu_168814.jpg "Optional title")

&emsp;&emsp;鉴于许多硬件设备厂商不希望公开其设备驱动的源代码，如果能将android的应用框架层与linux系统内核的设备驱动隔离，使应用程序框架的开发尽量独立于具体的驱动程序，则android将减少对Linux内核的依赖。HAL由此而生，它是对Linux内核驱动程序进行的封装，将硬件抽象化，屏蔽掉了底层的实现细节。HAL规定了一套应用层对硬件层读写和配置的统一接口，本质上就是将硬件的驱动分为用户空间和内核空间两个层面；Linux内核驱动程序运行于内核空间，硬件抽象层运行于用户空间。

#### 系统运行库层
&emsp;&emsp;官方的系统架构图中，位于Linux内核层之上的系统运行库层是应用程序框架的支撑，为Android系统中的各个组件提供服务。系统运行库层由系统类库和Android运行时构成。

* 系统类库
系统类库大部分由C/C++编写，所提供的功能通过Android应用程序框架为开发者所使用。例如SQlite,WebKit,SSL都在会在日常开发中有用到

![Alt text](http://images.51cto.com/files/uploadimg/20101129/1011272.png "Optional title")

* 运行时
Android运行时包含核心库和Dalvik虚拟机两部分。
    - 核心库：核心库提供了Java5 se API的多数功能，并提供Android的核心API，如android.os，android.net，android.media等。
    - Dalvik虚拟机：Dalvik虚拟机是基于apache的java虚拟机，并被改进以适应低内存，低处理器速度的移动设备环境。Dalvik虚拟机依赖于Linux内核，实现进程隔离与线程调试管理，安全和异常管理，垃圾回收等重要功能。
    
Dalvik和标准Java虚拟机有以下主要区别：

* Dalvik基于寄存器，而JVM基于栈。一般认为，基于寄存器的实现虽然更多依赖于具体的CPU结构，硬件通用性稍差，但其使用等长指令，在效率速度上较传统JVM更有优势。
* Dalvik经过优化，允许在有限的内存中同时高效地运行多个虚拟机的实例，并且每一个Dalvik应用作为一个独立的Linux进程执行，都拥有一个独立的Dalvik虚拟机实例。Android这种基于Linux的进程“沙箱”机制，是整个安全设计的基础之一。
* Dalvik虚拟机从DEX（Dalvik Executable）格式的文件中读取指令与数据，进行解释运行。DEX文件由传统的，编译产生的CLASS文件，经dx工具软件处理后生成。
* Dalvik的DEX文件还可以进一步优化，提高运行性能。通常，OEM的应用程序可以在系统编译后，直接生成优化文件（.ODEX）； 第三方的应用程序则可在运行时在缓存中优化与保存，优化后的格式为DEY（.dey文件）。

&emsp;&emsp;这部分内容，即从android4.4开始就出现了ART（android runtime），但是这个ART并不是指这一节的主题，而是一种用来代替Dalvik的新型运行环境。当然在4.4的正式环境中用的还是Dalvik，真正开始用ART取代Dalvik是从android5.0开始的。（todo:针对这个改动，楼主会专门另开一个篇幅的文章去探究ART和Dalvik之间的区别）

另外这一节中有提到NDK,相信对于开发者而言SDK和NDK都是必要要接触和了解的东西，那么先从下图来看看sdk和ndk的关系。

![Alt text](http://img.blog.csdn.net/20160107235017221 "Optional title")

&emsp;&emsp;很显然地，ndk可以通过native code跨过使用dalvik runtime,直接调用到android内核资源，而sdk则需要在dalvik runtime环境下才能调用到内核资源。然而两者并不是各司其职，各不相关。android提供了JNI(Java native interface)使两者可以进行相互调用和通信。

#### 应用程序框架层

&emsp;&emsp;应用程序框架层提供开发Android应用程序所需的一系列类库，使开发人员可以进行快速的应用程序开发，方便重用组件，也可以通过继承实现个性化的扩展。

![Alt text](http://images.51cto.com/files/uploadimg/20101129/1011271.png "Optional title")

#### 应用层

&emsp;&emsp;Android平台的应用层上包括各类与用户直接交互的应用程序，或由java语言编写的运行于后台的服务程序。例如，智能手机上实现的常见基本功能 程序，诸如SMS短信，电话拨号，图片浏览器，日历，游戏，地图，web浏览器等程序，以及开发人员开发的其他应用程序。

&emsp;&emsp;将android的基本架构进行了一个总体的分析和罗列，我们可以发现，平时开发中最常接触和用到的一定是application层，但是我们也不难发现，一些application层应用到的东西都能在系统层找到对应的踪迹，例如sqlite,webkit,甚至alarm。他们是怎么从底层到达application层供我们日常开发所用，这个也是需要去了解和研究的。本篇文章的目的在开篇已经阐述过，是为了能更好地将自己的知识对号入座，并且去补充一些自己在某些层面上缺乏的知识，最终可以将自己的知识形成一个整体的体系结构。
