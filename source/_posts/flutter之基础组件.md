---
title: flutter之基础组件
copyright: true 
date: 2020-01-22 18:47:41
tags: [Flutter]
category: Flutter
author: Yangcy
img: https://cn.bing.com/th?id=OHR.SaltonSea_ZH-CN1265210111_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp
summary: 了解什么是Widget，MaterialApp,以及常用的基础组件
---
### Widget
#### 什么是widget
一切皆是widget，是flutter应用用户界面的基本构建单元，每个widget都与最终的用户界面有密切相关
* 抽象类继承了可诊断树(DiagnosticableTree)
* 一个常量构造函数，可选参数key
* 抽象方法createElement
* 静态方法canUpdate，通过runtimeType和key判断

#### 一个widget的定义如下
* 一个结构元素(按钮，菜单)
* 一个风格元素(字体，配色方案等)
* 布局(padding)
* 等等

#### widget的分类
主要需要了解的是StatelessWidget和StatefullWidget
* StatelessWidget:无状态的，AssetImage,Text...
* StatefulWidget:有状态的，Scrollable,Animatable..
* Widget的介绍
    - 用于描述Element的配置
    - Widge他作为用户界面的一部分是不会改变的，被加载进Element(控制底层的渲染树)
    - Widget本身不包含状态或者可变数据，通过StatefulWidget.createState可以关联一个State
    - 一个Widget可以被多次插入视图树中，并被加载进Element中
    - 变量key和runtimeType用来判断Widget是否改变，是则重新加载
* 构建 widget 的过程并不耗费资源，因为 Wiget 只是用来保存属性的容器。
* 无法获取Widget在屏幕上的位置和大小，因为Widget就是一张蓝图，他只是描述了底层渲染对象应该具有的属性

#### StatelessWidget
* 继承自Widget的抽象类
* 重写了createElement方法名创建了StatelessElement对象
* 一个build方法，创建一个Widget
* 内部没有保存状态，UI界面创建后不会发生改变

#### StatefulWidget
* 继承自Widget的抽象类
* 重写了createElement方法名创建了StatefulElement对象
* 一个createState方法，创建一个State
* 内部保存状态，调用setState方法，变更UI

#### State
* 泛型抽象类 T extend StatefulWidget
* State的流程
    -  launch->initState->didChangeDependencies->build->deactive->dispose->destroy
    -  didUpdateWidget->build
    -  initState：被插入到Widget树中被调用一次
    -  didChangeDependencies:当state的依赖对象发生变化时调用
    -  build:构建Widget时调用
    -  didUpdateWidget:Widget重新构建时调用
    -  deactive:当state对象被从树中移除调用
    -  dispose:当state对象从树中被永久移除时调用，一般在此回调时释放资源

#### Element
* Element是控件树上的实例
* Element 通过 mount 方法插入到 Element Tree 中，创建了RenderObject对象
* 

#### 树 Widget Tree, Element Tree ,RenderObject Tree
* Widget Tree -(createElement)> Element Tree -(createRenderObject)> RenderObject Tree
* Widget是为了描述Element需要的配置，负责创建Element,决定Element是否需要被更新
* Element表示Widget配置树的特定位置的一个实例，同时持有Widget和RenderObject，负责管理Widget的配置和RenderObject的渲染。Widget发生改变，didUpdateWidget->build不会重建E，Element，只会更新
* RenderObject表示渲染树的一个对象，负责真正的渲染工作，比如测量大小，位置绘制等都是由RendeObject完成

#### key
* 使用Key可以控制框架在Widget重建时与哪些其他Widget进行匹配
* 包含有LocalKey和globalKey
    - LocalKey：ObjectKey，UniqueKey，ValueKey(PageStrageKey)
    - GlobalKey:LabeledGlobalKey,GlobalObjectKey

### MaterialApp
Material应用是以MaterialApp Widget开始，主要封装了应用程序实现Material Design所需要的配置
* 构造函数
    - 路由
    - 主题
    - 本地化
    - 性能监控，调试

#### Scaffold（脚手架）
在构造函数中，可以看出有App头布局，body内容,抽屉栏等元素控件，与原生Android有相似
```
const Scaffold({
    Key key,
    this.appBar,
    this.body,
    this.floatingActionButton,
    this.floatingActionButtonLocation,
    this.floatingActionButtonAnimator,
    this.persistentFooterButtons,
    this.drawer,
    this.endDrawer,
    this.bottomNavigationBar,
    this.bottomSheet,
    this.backgroundColor,
    this.resizeToAvoidBottomPadding,
    this.resizeToAvoidBottomInset,
    this.primary = true,
    this.drawerDragStartBehavior = DragStartBehavior.start,
    this.extendBody = false,
    this.extendBodyBehindAppBar = false,
    this.drawerScrimColor,
    this.drawerEdgeDragWidth,
  })
```

#### Text
```
const Text(
    this.data, { // 必要参数
    Key key,
    this.style,  // 文字样式
    this.strutStyle,
    this.textAlign, // 文字居中
    this.textDirection,
    this.locale,
    this.softWrap,
    this.overflow,
    this.textScaleFactor,
    this.maxLines,  // 最初
    this.semanticsLabel,
    this.textWidthBasis,
  })
```

#### TextField
表单操作，输入用户名密码
```
const TextField({
    Key key,
    this.controller,
    this.focusNode,  // 焦点
    this.decoration = const InputDecoration(), // 设置输入样式
    TextInputType keyboardType,
    this.textInputAction,
    this.textCapitalization = TextCapitalization.none,
    this.style,
    this.strutStyle,
    this.textAlign = TextAlign.start,
    this.textAlignVertical,
    this.textDirection,
    this.readOnly = false,
    ToolbarOptions toolbarOptions,
    this.showCursor,
    this.autofocus = false,
    this.obscureText = false,  // 密码输入
    this.autocorrect = true,
    this.enableSuggestions = true,
    this.maxLines = 1,
    this.minLines,
    this.expands = false,
    this.maxLength,
    this.maxLengthEnforced = true,
    this.onChanged,
    this.onEditingComplete,
    this.onSubmitted,
    this.inputFormatters,  // 输入限制，手机号，数字 11位等
    this.enabled,
    this.cursorWidth = 2.0,
    this.cursorRadius,
    this.cursorColor,
    this.keyboardAppearance,
    this.scrollPadding = const EdgeInsets.all(20.0),
    this.dragStartBehavior = DragStartBehavior.start,
    this.enableInteractiveSelection = true,
    this.onTap,
    this.buildCounter,
    this.scrollController,
    this.scrollPhysics,
  }) 
```

#### Image
* AssetsImage:需要配置pubspec.yaml

#### BoxFit
图片的拉伸，填充控制

#### Icon
```
  const Icon(
    this.icon, {  // 设置Icons.add 系统提供
    Key key,
    this.size,
    this.color,
    this.semanticLabel,
    this.textDirection,
  })
```
