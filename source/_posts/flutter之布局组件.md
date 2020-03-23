---
title: flutter之布局组件
copyright: true
date: 2020-01-23 13:33:34
tags: [Flutter]
category: Flutter
author: Yangcy
img: https://cn.bing.com/th?id=OHR.SaltonSea_ZH-CN1265210111_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp
summary: 线性布局，弹性布局，层叠布局
---
### 线性布局
线性布局，指的是沿水平或者垂直方向排布子组件。Flutter中通过Row和Column来实现线性布局。
* 主轴和纵轴的区分，依赖于布局方向
    - 布局是水平方向，主轴就是水平方向(Main Axis)
    - 反之，主轴就是竖直方向

#### Row
```
  Row({
    Key key,
    MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,  // 子组件在水平方向上的对齐方式
    MainAxisSize mainAxisSize = MainAxisSize.max,  // 主轴占用的空间
    CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center, //纵轴方向上的对齐方式
    TextDirection textDirection,  // 文字方向
    VerticalDirection verticalDirection = VerticalDirection.down, // 表示row纵轴的对齐方式,down:自上而下，up:自下而上
    TextBaseline textBaseline,
    List<Widget> children = const <Widget>[],
  })
```

### 弹性布局
弹性布局允许子组件按照一定比例来分配父容器空间。Flutter中的弹性布局主要通过Flex和Expanded来配合实现

#### Flex
Flex组件可以沿着水平或者垂直方向排列子组件
```
Flex({
    Key key,
    @required this.direction,
    this.mainAxisAlignment = MainAxisAlignment.start,
    this.mainAxisSize = MainAxisSize.max,
    this.crossAxisAlignment = CrossAxisAlignment.center,
    this.textDirection,
    this.verticalDirection = VerticalDirection.down,
    this.textBaseline,
    List<Widget> children = const <Widget>[],
  })
```

#### Expanded
可以按照比例"扩伸"，Row、Column和Flex子组件所占空间
```
const Expanded({
    Key key,
    int flex = 1,
    @required Widget child,
  }) 
```


### 层叠布局
层叠布局能够将子控件层叠排列。Flutter中Stack允许子控件堆叠，而positioned用于根据Stack的四个角来确定子组件的位置

#### Stack
```
  Stack({
    Key key,
    this.alignment = AlignmentDirectional.topStart, // 对齐方式
    this.textDirection,
    this.fit = StackFit.loose, // 如何占满Stack
    this.overflow = Overflow.clip, // 超出部分的显示
    List<Widget> children = const <Widget>[],
  })
```

#### Positioned
分别表示离Stack的 上下左右的的间距， 以及指定元素的大小
```
const Positioned({
    Key key,
    this.left,
    this.top,
    this.right,
    this.bottom,
    this.width,
    this.height,
    @required Widget child,
  })
```
