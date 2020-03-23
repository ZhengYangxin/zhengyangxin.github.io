---
title: flutter之动画
copyright: true
date: 2020-01-23 21:53:59
tags: [Flutter]
category: Flutter
author: Yangcy
img: https://cn.bing.com/th?id=OHR.SaltonSea_ZH-CN1265210111_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp
summary: Flutter动画的基本概念和动画结构
---
#### 基本概念
* Animation：Flutter动画库中的核心类，插入用于指导动画的值
* AnimationController:管理Animation
* CurvedAnimation:定义动画的曲线
* Tween:为动画对象插入一个范围值

#### Animation
* 提供了每一帧动画变化的监听事件和移除事件，VoidCallback
    - addListener
    - removeListener
* 提供了动画状态的监听和移除事件，AnimationStatusListener
    - addStatusListener
    - removeStatusListener
* 动画的四种状态
    - dismiss，在动画开始时停止
    - forward，动画从开始向结束运行
    - reverse，动画从结束向开始运行
    - completed，动画在结束时运行完成
* 获取动画的状态
* 泛型参数为范围值，一般是double类型

#### AnimationController
* 继承自Animation，是一个特殊的Animation，当硬件准备新帧时，它都会生成一个新值
* 需要一个vsync参数，vsync的存在防止后台动画消耗不必要的资源
```
AnimationController({
    double value,
    this.duration, // 动画时长
    this.reverseDuration,
    this.debugLabel, 
    this.lowerBound = 0.0,  // 最小值
    this.upperBound = 1.0, // 最大值
    this.animationBehavior = AnimationBehavior.normal,
    @required TickerProvider vsync,  // 每一帧同步时的回调
  })
```

#### CurvedAnimation
怎么运动，运动的过程，动画运行的曲线，可以继承Curve,重写transformInternal
* Curve曲线分类
    - linear:匀速的
    - decelerate：匀减速
    - ease：开始加速后面减速
    - easeIn：开始慢后面快
    - easeOut：开始快，后面慢
    - easeInOut：开始慢，先加速，后减速

#### Tween
可以自定义移动的类型，如移动像素等
* 配置动画插入不同的范围和数据类型

#### AnimatedBuilder
动画的构建器
```
const AnimatedBuilder({
    Key key,
    @required Listenable animation, // 具体动画
    @required this.builder, // 实现动画的对象
    this.child,
  })
```

#### 路由切换动画
* Android提供默认的MaterialPageRoute
* IOS提供默认的CupertionPageRoute
* 自定义实现采用PageRouteBuilder
```
Navigator.push(context, PageRouteBuilder(pageBuilder: (context, animation, econdaryAnimation){
  return FadeTransition(opacity: animation, child: SyncAnim(),);
}));
```

#### Hero动画
指在页面之间飞行的Widget，相当于转场动画，Hero在动画切换的时候，有一个共享的Widget可以在新旧路由之间切换
* InkWell:水波纹效果的widget
* 将需要共享的元素放入Hero Widget中
* 需要指定相同的tag
```
InkWell(
            child: Hero(tag: "avator", child: ClipOval(
              child: Image.network("http://yangxin.online/images/head.jpeg", width: 50, height: 50 , ),
            )),
            onTap: (){
              Navigator.of(context).push(MaterialPageRoute(builder: (context){
                return Scaffold(
                  body: Center(
                    child: Hero(tag: "avator", child: Image.network("http://yangxin.online/images/head.jpeg", width: 300, height: 300 , ),
                  ),
                ));
              }));
            },
          )
```

#### 交织动画
设计复杂动画，动画序列，重叠动画
* 使用多个动画对象
* 一个AnimationControl控制所有的动画
* 每个动画对象指定间隔时间
```
class Stagger extends StatelessWidget {
  final AnimationController controller;

  final Animation<double> opacity, width, height;

  final Animation<EdgeInsets> padding;

  final Animation<BorderRadius> borderRadius;

  final Animation<Color> color;

  Stagger({Key key, this.controller})
      : opacity = Tween(begin: 0.0, end: 1.0).animate(CurvedAnimation(
            parent: controller,
            curve: Interval(0.0, 0.1000, curve: Curves.linear))),
        width = Tween(begin: 50.0, end: 150.0).animate(CurvedAnimation(
            parent: controller,
            curve: Interval(0.125, 0.250, curve: Curves.linear))),
        height = Tween(begin: 50.0, end: 150.0).animate(CurvedAnimation(
            parent: controller,
            curve: Interval(0.250, 0.375, curve: Curves.linear))),
        padding = EdgeInsetsTween(
                begin: EdgeInsets.only(bottom: 10),
                end: EdgeInsets.only(bottom: 50))
            .animate(CurvedAnimation(
                parent: controller,
                curve: Interval(0.250, 0.375, curve: Curves.linear))),
        borderRadius = BorderRadiusTween(
                begin: BorderRadius.circular(5), end: BorderRadius.circular(15))
            .animate(CurvedAnimation(
                parent: controller,
                curve: Interval(0.375, 0.500, curve: Curves.linear))),
        color = ColorTween(begin: Colors.blue, end: Colors.red).animate(
            CurvedAnimation(
                parent: controller,
                curve: Interval(0.500, 0.750, curve: Curves.linear))),
        super(key: key);

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: controller,
      builder: (context, child) {
        return Container(
          padding: padding.value,
          alignment: Alignment.bottomCenter,
          child: Opacity(
            opacity: opacity.value,
            child: Container(
              width: width.value,
              height: height.value,
              decoration: BoxDecoration(
                  color: color.value,
                  border: Border.all(color: Colors.blue, width: 3),
                  borderRadius: borderRadius.value),
            ),
          ),
        );
      },
    );
  }
}

class StaggerF extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    // TODO: implement createState
    return StaggerS();
  }
}

class StaggerS extends State<StaggerF> with TickerProviderStateMixin {
  AnimationController _controller;

  _play() async {
    await _controller.forward().orCancel;
    await _controller.reverse().orCancel;
  }

  @override
  void initState() {
    super.initState();
    _controller =
        AnimationController(duration: Duration(seconds: 10), vsync: this);
    _controller.addStatusListener((status) {
      switch (status) {
        case AnimationStatus.dismissed:
          _controller.forward();
          break;
        case AnimationStatus.completed:
          _controller.reverse();
          break;
        case AnimationStatus.forward:
        case AnimationStatus.reverse:
          break;
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
      body: GestureDetector(
        onTap: () {
          _play();
        },
        child: Center(
          child: Container(
            width: 300,
            height: 300,
            color: Colors.yellow,
            child: Stagger(
              controller: _controller,
            ),
          ),
        ),
      ),
    );
  }
}
```
