---
title: flutter之路由
copyright: true
date: 2020-01-23 18:39:12
tags: [Flutter]
category: Flutter
author: Yangcy
img: https://cn.bing.com/th?id=OHR.SaltonSea_ZH-CN1265210111_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp
summary: 路由，页面跳转
---
### 路由管理
在Flutter中，屏于页面都叫做路由。路由管理，就是管理页面之间如何跳转，在Flutter中维护一个路由栈，路由入栈操作对应着就是打开一个新页面，路由出栈操作就是对应页面的关闭操作
* Navigator.push()跳转到第二个页面
* Navigator.pop()退回到第一个页面
```
 Navigator.push(context, MaterialPageRoute(builder: (context) {
    return SecondRoute2("hello word");
}));

Navigator.pop(context);

```

#### 路由传值
* Navigator.push()返回Flutter对象，用以接收新路由出栈时的返回数据
* Navigator.pop()将栈顶路由出栈，参数Result为页面关闭时返回给上一个页面的数据
```
 Scaffold(
      appBar: AppBar(
        title: Text("SendRoute route"),
      ),
      body: Center(
        child: Column(
          children: <Widget>[
            Text(arg),
            RaisedButton(
              child: Text(
                "go back",
              ),
              onPressed: () {
                Navigator.pop(context, "hello world");
              },
            ),
          ],
        ),
      ),
    );
```

#### 命名路由
* 注册路由表
* Navigator.pushNamed()跳转到第二个界面。
* Navigator.pop()回退到第一个路由
```
class MyApp2 extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      routes: {
        '/': (context) => FirstRoute(),
        '/second_route': (context) => SecondRoute()
      },
      initialRoute: '/',
    );
  }
}

// 跳转
Navigator.pushNamed(context, '/second_route', arguments: '卡啦啦啦');

// 获取命名跳转的传值
final String arg = ModalRoute.of(context).settings.arguments;


// 优先级最低路由列表
onGenerateRoute: (settings) {
        if(settings.name == '/second_route4'){
          var arg = settings.arguments;
          return MaterialPageRoute(builder: (context) => SecondRoute4(arg), settings: settings);
        }
      },

```

#### 路由动画
* MaterialPageRoute：继承自PageRoute，是MaterIAL组件库提供的组件，它可以针对不同的平台，实现与平台页面切换动画一致的路由切换动画
* PageRouteBuilder
```
  Navigator.push(context, PageRouteBuilder(
    transitionDuration: Duration(milliseconds: 800),
    pageBuilder: (BuildContext context, Animation<double> animation, Animation<double> secondaryAnimation){
      return FadeTransition(opacity: animation, child: SecondRoute5(),);
    }
  ));
```
