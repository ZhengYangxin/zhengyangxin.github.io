---
title: Android IPC
copyright: true
date: 2018-03-31 10:39:30
tags: [Android, 进程通讯]
category: Android
---
### Android跨进程通讯的几种方式
#### Bundle
  Bundle实现了Parcelable，方便在进程中传输数据。主要在activity、service、receiver中Intent中应用，
#### 文件共享
Android基于Linux，对文件的读写没有限制。存在的问题就是并发读写的问题
sp存在缓存策略，内存中存在备份，导致多进程不可靠
#### 使用messenger信使
1. Messenger可以在不同的进程中传递Message对象，轻松实现数据的跨进程通讯。
2. 它的底层实现是AIDl，内部一次只做一次处理，因此服务端不用考虑线程同步的问题
3. 缺点是只能用来简单的信息传递，并发请求不大合适且无法不支持跨进程的方法的调用
![Alt text](http://opd2n5pxb.bkt.clouddn.com/Messenger%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%9B%BE.png "Messenger 工作原理图")

