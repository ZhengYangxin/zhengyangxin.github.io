---
title: flutter之列表控件
copyright: true
date: 2020-01-23 15:27:57
tags: [Flutter]
category: Flutter
author: Yangcy
img: https://cn.bing.com/th?id=OHR.SaltonSea_ZH-CN1265210111_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp
summary: ListView，GridView
---
#### ListView
* 少量数据
```
ListView({
    Key key,
    Axis scrollDirection = Axis.vertical, // 滚动方向
    bool reverse = false, // 是否反向展示数据
    ScrollController controller,
    bool primary,
    ScrollPhysics physics, // 物理滚动，默认根据不同平台采用不同对象
    bool shrinkWrap = false,
    EdgeInsetsGeometry padding,
    this.itemExtent, // item 有效范围
    bool addAutomaticKeepAlives = true, // 自动保存视图缓存
    bool addRepaintBoundaries = true, // 添加重绘边界
    bool addSemanticIndexes = true,
    double cacheExtent,
    List<Widget> children = const <Widget>[],
    int semanticChildCount,
    DragStartBehavior dragStartBehavior = DragStartBehavior.start,
  })
```

* 长列表数据
    - itemBuilder:它是列表项的的构造器，返回值是一个Wideget，当列表滚动到具体的index时，会调用改构造器构建列表
    - itemCount：列表项的数量，数量为null，则为无限列表
```
ListView.builder({
    Key key,
    Axis scrollDirection = Axis.vertical,
    bool reverse = false,
    ScrollController controller,
    bool primary,
    ScrollPhysics physics,
    bool shrinkWrap = false,
    EdgeInsetsGeometry padding,
    this.itemExtent,
    @required IndexedWidgetBuilder itemBuilder,
    int itemCount,
    bool addAutomaticKeepAlives = true,
    bool addRepaintBoundaries = true,
    bool addSemanticIndexes = true,
    double cacheExtent,
    int semanticChildCount,
    DragStartBehavior dragStartBehavior = DragStartBehavior.start,
  })
```

* 分割组件生成器
```
ListView.separated({
    Key key,
    Axis scrollDirection = Axis.vertical,
    bool reverse = false,
    ScrollController controller,
    bool primary,
    ScrollPhysics physics,
    bool shrinkWrap = false,
    EdgeInsetsGeometry padding,
    @required IndexedWidgetBuilder itemBuilder,
    @required IndexedWidgetBuilder separatorBuilder,
    @required int itemCount,
    bool addAutomaticKeepAlives = true,
    bool addRepaintBoundaries = true,
    bool addSemanticIndexes = true,
    double cacheExtent,
  })
```

#### GridView
```
GridView({
    Key key,
    Axis scrollDirection = Axis.vertical,
    bool reverse = false,
    ScrollController controller,
    bool primary,
    ScrollPhysics physics,
    bool shrinkWrap = false,
    EdgeInsetsGeometry padding,
    @required this.gridDelegate,  // 表格处理类，SliverGridDelegateWithFixedCrossAxisCount，SliverGridDelegateWithMaxCrossAxisExtent
    bool addAutomaticKeepAlives = true,
    bool addRepaintBoundaries = true,
    bool addSemanticIndexes = true,
    double cacheExtent,
    List<Widget> children = const <Widget>[],
    int semanticChildCount,
  })
```