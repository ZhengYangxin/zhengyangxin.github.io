---
title: Handler解析
copyright: true
date: 2019-10-19 21:27:32
tags: [Handler, 源码分析]
category: "Android"
---
#### Handler的几个问题
1. Handler内存泄漏的测试
在消息未发送之前，将activity销毁，需要remove消息及将handler赋值为null
2. 为什么不能在子线程创建Handler
在子线程中直接new Handler，会获取当前线程的Looper，但我们并没有穿件Looper
3. textView.setText()只能在主线程执行？
是的，因为setText()会刷新界面，调用requestLayout及invalidate，内部会做是否为主线程的检查
4. new Handler()两种写法有什么区别？
直接new Handler 重写他的handlerMessage()方法不推荐，通过new Handler(new Handler.CallBack)是推荐做法 CallBack接口的handlerMessage方法应返回为true
5. ThreadLocal用法和原理
ThreadLocal 内部是一个Map对象，保存着key和value，可以为当前的线程，value为线程保存的值

#### 源码分析
1. 从应用的创建：ActivityThread.main() -> Looper.preMainLooper-> 这时候ThreadLocal.get() = null ->会创建主线程的Looper对象及唯一关联的MessageQueue
2. 当在主线程中new Handler()时 -> 内部回调Looper.myLooper获取到主线程的Looper-> 将looper的messagequeue赋值给Handler的变量 -> 当handler调用sendMessage -> queue.enqueueMessage()
3. 在ActivityThread.main() -> 会调用Looper.loop()方法 -> 从消息队列中获取消息 ->
4. 调用message.target.dispachMessage -> target即Handler，则会调用handler中的dispatch方法 -> 最终掉用handler的handlerMessage方法

#### 解决几个问题
1. 为什么主线程用Looper死循环不会引发ANR异常
因为Looper.next开启死循环的时候，一旦需要等待时或者还没有执行到执行的时候，会调用NDK里的JNI方法，释放当前的时间片，这样就不会引发ANR异常
2. 为什么Handler构造方法里面的Looper不是直接new？
因为直接new出来不一定能保证唯一，只有用Looper.prepare才能保证唯一性
3. MessageQueue为什么要放在Looper的私有构造方法里初始化
因为一个线程只能由一个Looper，所以在构造方法里初始化也就能保证MessageQueue的唯一性
4. 主线程里的Looper.prepare/Looper.loop,是一直在无限循环里面的吗
是的

