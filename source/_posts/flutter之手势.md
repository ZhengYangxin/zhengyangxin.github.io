---
title: flutter之手势
copyright: true
date: 2020-01-25 22:11:11
tags: [Flutter]
category: Flutter
author: Yangcys
img: https://cn.bing.com/th?id=OHR.SaltonSea_ZH-CN1265210111_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp
summary: Flutter手势，控件与交互
---

### 指针
描述屏幕上由触摸板，鼠标，指示笔等触发的指针的位置和移动

#### 指针事件
* PointerDownEvent：指针在特定位置与屏幕接触
* PointerMoveEvent：指针从屏幕的一个尾椎移动到另一个位置
* PointerUpEvent：指针与屏幕停止接触
* PointerCancelEvent：指针的输入已经不在指向此应用

#### Listener
```
const Listener({
    Key key,
    this.onPointerDown,
    this.onPointerMove,
    // We have to ignore the lint rule here in order to use deprecated
    // parameters and keep backward compatibility.
    // TODO(tongmu): After it goes stable, remove these 3 parameters from Listener
    // and Listener should no longer need an intermediate class _PointerListener.
    // https://github.com/flutter/flutter/issues/36085
    @Deprecated(
      'Use MouseRegion.onEnter instead. See MouseRegion.opaque for behavioral difference. '
      'This feature was deprecated after v1.10.14.'
    )
    this.onPointerEnter, // ignore: deprecated_member_use_from_same_package
    @Deprecated(
      'Use MouseRegion.onExit instead. See MouseRegion.opaque for behavioral difference. '
      'This feature was deprecated after v1.10.14.'
    )
    this.onPointerExit, // ignore: deprecated_member_use_from_same_package
    @Deprecated(
      'Use MouseRegion.onHover instead. See MouseRegion.opaque for behavioral difference. '
      'This feature was deprecated after v1.10.14.'
    )
    this.onPointerHover, // ignore: deprecated_member_use_from_same_package
    this.onPointerUp,
    this.onPointerCancel,
    this.onPointerSignal,
    this.behavior = HitTestBehavior.deferToChild,
    Widget child,
  })
```

#### HitTestBehavior
* translucent：层叠布局时，可以使布局都收到事件
* opaque: 布局透明也可以收到事件
* deferToChild：当布局透明时收不到事件
```
enum HitTestBehavior {
  /// Targets that defer to their children receive events within their bounds
  /// only if one of their children is hit by the hit test.
  deferToChild, // 子组件会一个接一个的进行命中测试，如果子组件收到，那父组件也能收到该事件

  /// Opaque targets can be hit by hit tests, causing them to both receive
  /// events within their bounds and prevent targets visually behind them from
  /// also receiving events.
  opaque, // 在命中测试中，将当前组件当成不透明处理

  /// Translucent targets both receive events within their bounds and permit
  /// targets visually behind them to also receive events.
  translucent, // 当点击组件透明区域时，可以对自身边界内及底部可视区域，都进行命中测试
}
```

#### 忽略指针事件
* IgnorePointer：阻止子树接受指针事件，本身不会参与命中测试
* AbsorbPointer：阻止子树接受指针事件，本身参与命中测试

### 手势
Gesture代表的是语义操作（比如点击，拖动，缩放），通过一系列单独的指针事件组成，甚至是一系列指针组成。Gesture可以分发多种事件，对应着指针的生命周期（比如开始拖动，拖动更新，结束拖动）

#### 手势类别
* GestureDetector：用于手势识别的功能性组件，可以识别各种手势，是指针事件的语义分装，内部使用了一个或者多个GestureRecognizer
* GestureRecognizer：通过Listener将原始指针事件转化为语义手势

#### GestureDetector
* 点击：onTapDown，onTapUp，onTap，onTapCancel
* 双击：onDoubleTap
* 长按：onLongPress
* 纵向拖动：onVerticalDragStart，onVerticalDragUpdate，onVerticalDragEnd
* 横向拖动：onHorizontalDragStart，onHorizontalDragUpdate，onHorizontalDragEnd
* 移动：onPanStart,onPanUpdate,onPanEnd
* 移动和横向，纵向拖动互斥

#### 手势消歧处理
* 在屏幕的指定位置上，可能有多个手势捕捉器。所有的手势捕捉器监听了指针输入流事件并判断出特定的手势
* 在任何时候，识别器都可以宣告失败并离开竞技场。如果竞技场中只有一个识别器，那么这个识别器就是胜者。
* 在任何时候，任何识别器都可以宣告胜利，这将导致这个识别器胜出，其他识别器失败
