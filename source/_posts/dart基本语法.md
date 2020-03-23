---
title: dart基本语法
copyright: true
date: 2020-01-21 13:04:23
tags: [Dart, Flutter]
category: Flutter
author: Yangcy
img: https://cn.bing.com/th?id=OHR.SaltonSea_ZH-CN1265210111_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp
summary: Dart的基本语法，运行环境安装等
---
### Dart的开发环境安装
#### 概述
1. Dart可以用来开发移动应用，web应用，开发命令行应用和服务端应用，主要有以下IDE可供选择
2. VS Code:在其中安装Dart插件进行开发
3. Android Studio：主要用于移动开发
4. Web Storm,IntelliJ, DartPad在线运行

#### Dart SDK概要
1. Dart SDK包含开发web，命令行和服务端应用所需要的库和命令行工具。如果是需要开发移动应用，只需要安装flutter即可

#### Dart SDK安装（homebrew）
1. install
```
brew tap dart-lang/dart
brew install dart
```

#### Dart SDK升级
1. update
```
brew upgrade dart
```
2. 检查是否安装成功
```
dart --version
```

#### 设置环境变量
```
vi .bash_profile

// click e 进入编辑模式
export Path=${PATH}:dart的安装目录
```

#### VS Code的开发环境
1. VS Code的下载进行安装
2. Dart的环境配置,在VS extesion中搜索并下载dart插件

#### DartPad 在线运行
1. 打开dartpad.cn可以直接运行代码，但是如果有外部包导入，则需要VS Code

### Dart基本概念
#### 样例程序
```
// 定义一个函数
printInteger(int aNumber){
  print('The number is $aNumber');
}

// Dart 程序从 main()函数开始执行
void main(){
  var number = 42; // 声明并初始化一个变量
  printInteger(number); // 调用一个函数
}
```
1. // 表示注释
2. int 表示数据类型
3. main 顶级函数，应用程序的入口
4. var 用于定义变量，可以不指定变量类型

#### 重要概念
1. 一切皆对象：所有变量引用的都是对象，数字，函数，null都是对象，都继承字Object类
2. Dart声明变量类型可选：Dart可以进行类型推断，dynamic可以声明一个不确定的类型
3. Dart支持泛型：List<int>或者List<dynaamic>(由任何类型对象组成的列表)
4. Dart支持顶级函数，支持属于类或者对象的函数，支持嵌套函数：main
5. Dart支持顶级变量，支持属于类或者对象的变量
6. 标识符下划线开头表示库内私有：_number,_name()
7. 标识符字母，数字，下划线，由字母或者下划线开头
8. Dart表达式有值，语句没有值
9. Dart工具可以显示警告和错误两种类型

#### 关键字
分为1，2，3
* 1表示上下文关键字，在特定的场合才有用
* 2表示内置关键符
* 3是1.0之后支持异步的关键字

#### 变量
1. 变量仅存储对象的引用
2. 变量声明的时候可以不指定类型
3. 未初始化的变量内容都为null
4. 可以使用关键字final或者const修饰变量 final只能赋值一次，const为编译时常量，顶层的final变量或者类的final变量在其第一次使用的时候初始化

### Dart内置类型
#### int 
* 长度不超过64位，具体取值范围依赖于不同的平台。在DartVM上其取值位于-2^63至2^63-1.编译吃JavaScript的Dart使用JavaScript数字，范围是-2^53~2^53-1之间

#### double

#### String
* Dart字符串是UTF-16编码的字符序列。可以使用单引号或者双引号创建字符串
* 可以使用+运算拼接字符串
* 使用三个单引号或者三个双引号穿件多行字符串
* 字符串前加上r作为前缀创建“raw”字符串（不会被做任何处理）
```
  
  var s1 = "dsadsa";
  var s2 = 'dadasd';
 
  var s3 = 'dasda\'';

  var s4 = "ab" + "cd";

  var s5 = "dsad""dasda";

  var s6 ='''
  dasfsa
  dsadas
  ''';

  var s8 = "dasdas is $a";

  const s10 = "a const";
```

#### Booleans
* bool关键字表示布尔类型，布尔类型只有true和false，是编译时常量
* Dart的类型安全不允许使用1,0做代码判断

#### Lists
* Dart中数组由List对象表示的
* 下标从0开始
* List list = List();//固定长度为数组，无参表示可变长度
```
  var list = [1,2,3];
  List list1 = new List();
  List list2 = List();
  list2.addAll(list);

  var temp = list2[0];

  var list3 = [0, ...list2];

  var list4 = [0, ...?list3];

  const list5 = [0,1,2]; 

  // list5[1] = 1; //  不能修改
```

#### Sets
* Dart中使用sets表示元素无序，唯一的值
* 支持Set字面量以及Set类型两种形式的set
* Set字面量是在Dart2.2中加入的
```
 var set1 = {'1'};
  var set2 = <int>{}; // 空的set， 不指定类型可用<dynamic>{}
  Set<int> set3 = Set();

  const set4 = {'a'};a
```

#### Maps
* Dart中的Map通过map字面量和map类型来实现
* 每个键只能出现一次，但是值可能出现重复
```
 var map = {1:"a", 2:"b"};
  Map map1 = Map();

  Map<int, String> map2 = Map();

  map2[0]; //0 是可以不是下标

  final  map3 = const {1:"a", 2:"b"};
```

#### Runes
* dart使用Runes来标识UTF-32编码的字符串
* String类中的codeUniteAt和codeUnite属性返回16位代码单元。Runes属性可以获取字符串的Runes

#### Symbols
* Symbols表示Dart中声明的操作符或者标识符，该类型的对象几乎不会被使用到
* 可以使用在标识符前面加#来获取Symbols
* Symbols字面量是编译时的常量

### Dart方法
 Dart是一种真正的面向对象的语言，所以函数也是对象并且类型为Function，这意味着函数可以被赋值给变量或者作为其他函数的参数。可以像调用函数一样调用Dart类的实例

 #### 参数
 * 函数可以有两张形式的参数；必要参数和可选参数
 * 必要参数定义在参数列表的前面
 * 可选参数定义在必要参数的后面
 * 可选参数
     -  可选参数分为命名参数和位置参数
     -  可选参数列表中任选其一使用，不能混用

```
int a(int a, int b,{int c, int d = 0, int f}){
  return a+b;
}

void main(){
  int c = a(2,3, c:4, d: 6);
}
// 使用参数名:参数值，的形式来指定命名参数
// 使用大括号的来指定命名参数
// 可以提供默认值
// @required注解来标识一个命名参数是必须的


int a(int a, int b,[int c, int d = 0, int f]){
  return a+b;
}
// 使用中括号将一系列参数包裹起来作为位置参数
// 可以使用=为函数的位置参数设置默认值，默认值必须是常量默认是null

```

#### main()函数
* 每个Dart程序都必须有一个main()顶级函数作为程序入口

#### 函数作为一级对象
* 可以将函数作为参数传递给另一个函数
* 可以将函数赋值给另一个变量

```
var f=  printE;
  var a = (e) => "dsada "; // 胖箭头语法

  var b = (e){
    return "xxx";
  };

```

#### 匿名函数
* 没有名字的函数
```
([[类型] 参数[,..]]){
    函数体;
}
```

#### 词法作用域
* 变量的作用于在写代码的时候就确定了
* 大括号内定义的变量只能在大括号内使用

#### 词法闭包
* 闭包即一个函数对象，即使函数对象的调用在它原始作用于范围之外，依然可以访问在它词法作用域内的变量

#### 返回值
* 所有函数都有返回值
* 没哟显示返回语句的函数，默认返回 return null

#### 运算符

### 流程控制语句
#### if else
* if else
* 三目运算 a == null? "guest": a;
* a ?? "guest"

#### for 循环语句
```
  var listaa =['a', 'q', 'f'];

  for (var item in listaa) {
    
  }

  for (var i = 0; i < listaa.length; i++) {
    
  }

  listaa.forEach((f){

  });
```

#### while /do while

#### switch

#### break continue

### 异常
* Dart能够Throw和catch异常，
* Dart中的所有异常为非检查异常，方法不一定声明他们所抛出的异常，并且你也不需要捕获异常
* Dart提供了Exception和Error类型，以及一些子类，也可以实现自己的异常类型

#### 抛出异常
```
throw FormatException("Excepted at least 1 selection");

// 任意类型的异常对象
throw "out of IIams";
```

#### 捕获异常
```
try {

} on XXXException {

} on Exception catch(e) {

} catch(e, s) {

} finally {

}
```

### 类
Dart是一个面向对象的编程语言，同时支持基于mixin的继承机制。每个对象都是一个类的实例，所有的累都继承Object。基于Mixin的继承意味着每个类都只有一个超类，一个类的代码可以在其他多个类继承
中重复使用

#### 使用类的成员
* 对象的成员由函数和数据（即方法和实例变量）组成，使用(.)来访问对象的实例变量或者方法

#### 使用构造函数
* 可以使用构造函数来创建一个对象。构造函数的命名方式可以为类名或者类名.标识符的形式

```
  var p = Point();
  var p2 = Point.fromJson({'x': 1, 'y': 2});
  p.x = 3; // 使用x的setter方法
```

#### 实例变量
```
class Point{
  int x; //声明变量x并初始化为null
  int y;
  num z = 0; //声明变量z并初始化为0
}

void main(){
  var p = Point();
  // var p2 = Point.fromJson({'x': 1, 'y': 2});

  p.x = 3; // 使用x的setter方法

  assert(p.x == 3);// 使用x的getter方法
  assert(p.y == null); //默认值为null
}
```

#### 构造函数
* 声明一个与类名一样的函数，即可声明一个构造函数
* 对于大多数编程语言来说在构造函数中为变量赋值的的过程都类似，而Dart则提供了一种特殊的语法糖简化该步骤
```
class Point{
  int x; //声明变量x并初始化为null
  int y;
  num z = 0; //声明变量z并初始化为0

  // Point(int x, int y){
  //   this.x = x;
  //   this.y = y;
  // }

  // this.x,this.y, Dart特殊的语法糖构造函数赋值
  Point(this.x, this.y);
}
```
* 默认构造函数：如果没有声明构造函数，Dart会自动生成一个无参的构造函数，并且该构造函数会自动调用其父类的无参构造函数
* 构造函数不会被继承：子类不会继承父类的构造函数
* 命名式构造函数：可以为一个类声明多个命名式的构造函数来表达更明确的意图
```
class Point{
  int x; //声明变量x并初始化为null
  int y;
  num z = 0; //声明变量z并初始化为0

  // 命名式构造函数
  Point.origin(){
    this.x = 0;
    this.y = 0;
  }
}
```
* 重定向构造函数:有时候类中的构造函数会调用类中其他的构造函数，该重定向构造函数没有函数体，只需要在函数签名后面使用（:）指定需要重定向到的其他构造函数既可以
```
  // 委托实现给主构造函数
  Point.alongXAxis(int x):this(x, 0);
```
* 常量构造函数：如果类生成的对象都是不变的，那么可以在生成这些对象时就将其变为编译时常量，你可以在类的构造函数前加上 const 关键字并确保所有实例均为final来实现该功能
```
class ImmutablePoint{
  static final ImmutablePoint point = const ImmutablePoint(0, 0);
  final int x, y;
  const ImmutablePoint(this.x, this.y);
}
```
* 工厂构造函数: 使用factory关键字表示类的构造函数将会令该构造函数变为工厂构造函数，这将意味着使用该构造函数构造类的实例时并非总是先返回新的实例对象
```
class Logger {
  final String name;

  static final Map<String, Logger> _cache = <String, Logger>{};
  factory Logger(String name){
    return _cache.putIfAbsent(name, () => Logger._internal(name));
  }

  static Logger _internal(String name){
    return Logger(name);
  }
}
```

#### 初始化列表
* 在构造函数体执行之前初始化实例变量
```
  Point.fromJson(Map<String, int> json){
    x = json['x'];
    y = json['y'];
    print("fromJson(): ($x, $y)" );
  }
```

#### 调用父类构造函数
* 构造函数调用顺序
    - 初始化列表
    - 父类的无参构造函数
    - 当前类的构造函数
* 如果父类无无参构造函数，那么子类必须调用父类的其中一个构造函数，为子类的构造函数指定父类的构造函数只需要在构造函数体前使用(:)指定

#### 方法
* 实例方法：实例方法可以访问实例的变量和this
* getter和setter的方法
* 抽象方法：定义一个借口方法而不去做具体的实现让实现他的类去实现该方法，抽象方法只能存在与抽象类中

#### 抽象类
* 使用关键字abstract标识的类让类成为抽象类，抽象类将无法被实例化。抽象类常用于声明接口方法，有时候也会有具体的实现方法
* 抽象类尝尝会包含抽象方法

#### 扩展类
* 继承 extend
* 子类可以重写父类的实例方法，getter,setter方法

#### 枚举类型
是一种特殊的类型，用于存储一些固定数量的常量
* 使用关键字 enum
* 每一个枚举值都有一个名为index成员变量的getter方法
* 使用枚举类的values方法获取一个包含所有枚举值的列表
* switch中使用枚举

#### 使用mixin为类添加功能
mixin是一种在多继承中复用某各类中代码的方法模式
* 定义一个类继承自Object并且不为该类定义构造函数，这个类就是Mixin类，通过关键字mixin替换class让其成为一个单纯的Mixin类
* 使用with关键字并在其后面跟上Mixin类的名字来使用Mixin模式

#### 静态变量和方法
static 关键字修饰

### 泛型
#### 为什么使用功能泛型
* 正确使用泛型可以生成更好的代码
* 使用泛型减少重复代码
* 构造方法时也可以使用泛型，在类名后用尖括号<..>将一个或多个类型包裹
```
var nameSet = Set<String>.from(names);
var views = Map<int, View>();
```
* Dart的泛型类型是固化的，这意味着即便在运行时也会保持类的信息（java中的泛型是类型擦拭的）

#### 限制参数化类型
* 使用extends关键字限制
