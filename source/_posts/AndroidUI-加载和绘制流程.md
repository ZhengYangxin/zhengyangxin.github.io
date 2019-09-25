---
title: AndroidUI 加载和绘制流程
copyright: true
date: 2019-09-25 13:24:26
tags: [学习]
category: "AndroidUI"
---
### 源码总结
#### View是如何被添加到屏幕窗口上去的
1. 在Activity的onCreate方法中调用setContentView将我们的布局传入
2. 内部调用了Window的唯一实例对象PhoneWindow的setContentView方法
3. 内部创建了顶层布局容器DecorView
4. 在DecorView中加载了基础布局ViewGroup(主题布局)，
5. 将我们的布局添加到基础布局的FrameLayout中 

#### View的绘制流程
1. 绘制入口
    * ActivityThread.handlerResumeActivity
    * WindowManagerImpl.addView(decorView, layoutParams)
    * WindowManagerGlobal.addView()
2. 绘制的类及方法
    * ViewRootImpl.addView(decorView, layoutParams, parentView)
    * ViewRootImpl.requestLayout()->scheduleTraversals()->doTraversals()->performTraversals()
3. 绘制流程的三大步骤
    * ViewRootImpl.performMeasure()
    * ViewRootImpl.performLayout()
    * ViewRootImpl.performDraw
4. 绘制中的测量
    * MeasureSpec的确定及计算: 顶层DecoView和其他View
    * DecorView通过窗口大小和DecoView本身大小确定
    * 其他View通过父View的MeasureSpec和本身的大小确定
5. 绘制中的布局
    * view.layout的方法确定自身的位置
    * 若是ViewGroup类型，需要调用onLayout的方法确定子View的位置
6. 绘制中的绘制
    * 绘制背景drawableBackground
    * 绘制自己onDraw
    * 绘制子View  dispatchDraw
    * 绘制前景，滚动条装等饰

#### 最后
onMeasure -> onLayout(ViewGroup要实现) -> onDraw(可选，系统控件不用实现)
