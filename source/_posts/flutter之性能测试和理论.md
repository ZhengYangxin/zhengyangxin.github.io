---
title: flutter之性能测试和理论
copyright: true
date: 2020-01-28 15:07:54
tags: [Flutter]
category: Flutter
author: Yangcy
img: https://cn.bing.com/th?id=OHR.SaltonSea_ZH-CN1265210111_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp
summary: flutter性能测试，理论
---
#### Flutter的渲染阶段
**VSync->Animation->Build->Layout->Paint->Display List(GPU)**
* Build ： Widget，Element树
* Layout，Paint： RenderObject树的创建
* Display List(GPU)，是对Layer图层的创建

#### 检查Flutter的渲染
Debug模式和最终的生产模式，有很大的性能特点，所以在做真实的测量之前都用Profile模式
* Debug模式
* Profile模式

#### RenderObject
它不是一个顶层Api，并且充分利用了组合的性质，所以他的公共方法比Android的View少
* 是Flutter的UI单位，有很长的生命周期和状态
* 主要方法
    - createRenderObject、updateRenderObject
    - performLayout
    - paint

#### Element
* ComponentElement，主要是做组合的，不直接参与布局绘制
    - StatelessElement
    - StatefulElement
* RenderObjectElement，会对RenderObject树上的RenderObject做连接

#### 同类型更新
修改某个Text的值
* Build
    - 在Flutter中Widget树是不可改变的，这也包括树节点之间的父子关系
    - 在开始build时，会创造一个新的树
    - 遍历Element，通过Element.updateChild(),观察子节点，若子节点类型发生改变，则会扔掉老节点，创造一个新的节点
    - 更新过程中，若Element是Component则build即可，若是RenderObjectElement则会updateRenderObject
    - 若内容发生改变会进行标脏处理

#### Buid性能测试工具
* 观望台，TimeLine，类似安卓的systrace,设置debugProfileBuildsEnable = true,检查build过程的性能损耗
* 遍历的触发 
    - setState方法
    - 依赖了InHeritedWidget,当InHeritedWidget发生改变会影响其他
    - 热重载，所有节点都会被更新
* 提高build效率
    - 通过Extra单独的Widget，减少遍历的节点
    - 停止遍历

#### Paint
在工程完成后会对RenderObject的某些节点进行标脏，让他重新绘制。
debugProfileBuildsEnabled = true;
debugProfilePaintsEnabled = true;
debugPaintLayerBordersEnabled = true;
* 如何知道多少其他节点需要跟被标脏的树一起被更新呢？
    - 基于图层树Layer
    - 更新指定图层
* Layer种类
    - PictureLayer
    - ContainerLayer:主要用于做PictureLayer的连接

yum -y install wget
wget -N –no-check-certificate https://softs.fun/Bash/ssr.sh && chmod +x ssr.sh && bash ssr.sh


