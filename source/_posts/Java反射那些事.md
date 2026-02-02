---
title: Java反射那些事
copyright: true
date: 2018-01-29 11:05:10
tags: 【反射机制】
category: Java
---
## 反射机制的定义
在运行状态期间，能过动态的知道一个类的属性和方法，能够动态的调用一个对象的属性和方法的功能。
## 反射机制的功能
在运行期间
1. 判断任意一个对象所属的类
2. 构建任意一个类的对象
3. 判断任意一个类的属性和方法
4. 调用任意一个对象的方法
5. 生成动态代理
<!-- more -->
## 反射机制的应用场景
1. 逆向代码，反编译
2. 与注解相结合的框架，retrofit
3. 单纯的反射机制框架，EventBus
4. 动态生成类框架，Gson

## 通过反射获取类信息
每个类被加载后，系统会为该类生成一个对应的Class对象，通过该Class对象就可以访问JVM中的这个类

#### 在JAVA程序中获得Class对象通常有三种方法
1. 使用Class类的forName()静态方法,传入全限定名
2. 调用某个类的class属性 
3. 通过对象getClass()获取Class对象
```
1.Classs  class = Class.forName(com.zyx.Person);
2.Class calss = Person.class;
3.Person person = new Person();
Class class = person.getClass();
```
#### 获取class对象的属性、方法、构造函数
1. 获取class对象的属性
```
Field[] allFields = class.getDeclaredFileds();// 获取所有声明的属性
Filed[] publicFileds = calss.getFields[]; //获取所有public属性
Field ageField = class.getDeclaredFiled("age");// 获取指定声明的属性
Filed desFiled = calss.getField("age"); //获取指定public属性
```
2. 获取class对象的方法
```
Method[] methods = class.getDeclaredMethods();// 获取所有声明的方法
Method[] publicFileds = calss.getMethods[]; //获取所有public方法，
Method[] publicFileds = calss.getMethods[]; //获取所有public方法，带指定形参列表的方法
Method method= class.getDeclaredMethods("info",String.class);// 获取指定声明的方法，
Method infoMethod = calss.getMethod("info",String.class); //获取指定public方法,带指定形参列表的方法
```
3. 获取class对象的构造函数
```
Constructor
```
#### 通过Java反射生成并操作对象
**生成实例**
1. 使用Class对象的newInstance()方法来创建Class对象对应的实例，但对应类必须有默认的构造函数
2. 先使用Class的对象获取指定的Constructor对象，在调用Constructor对象的newInstance()方法
**调用方法**
1. 通过Class对象的getMethods()或者getMethod()获得指定方法，返回Method对象或者数组。
2. 通过Method对象中的invoke()方法。第一个参数传调用该方法的对象，第二个参数传对应该方法的参数。
```
Object obj = class.newInstance();
Method methdo = calss.getDeclareMethod("setAge", int.calss);
method.invoke(obj, 28); //会检查调用权限
```
**访问成员变量赋值**
```
Object obj = class.newInstance();
Field field = class.getField("age");
field.setInt(obj, 28);
int age = field.getInt(obj);
```

## 代理模式
#### 定义
给某个类提供一个代理对象，并由代理对象控制对原对象的访问，即客户不直接操控原对象，而是通过代理对象操控原对象
#### 代理模式的分类
1. 静态代理，在编译时就实现好了，会生成对应的Class实际文件
2. 动态代理，在运行时生成，在运行时生成类字节码被加载到JVM中

#### 代理模式的思路
1. 代理对象和目标对象均实现同一个行为接口
2. 代理对象和目标对象分别实现具体接口逻辑
3. 在代理对像的构造函数中实例化一个目标对象
4. 在代理对象中调用目标对象的行为接口
5. 客户端想要调用目标对象的行为接口只能通过代理对象来操作

#### Java反射机制和动态代理
**动态代理介绍**
1. 运行时生成代理类，并将代理类的字节码载入当前代理的ClassLoader
2. 不需要多些多写一个与目标类相同的代理类
3. 可以在运行时定制代理类的执行逻辑
**涉及的类**
1. java.lang.reflect.Proxy,生成代理类的主类，通过Proxy生成的代理类都继承Proxy。Proxy提供了创建动态代理类和代理对象的方法，是所有动态代理类的父类。
2. java.lang.reflect.InvacationHandle 调用处理器，当调用动态代理的方法时会直接转到InvocationHandle的invoke()方法

#### 泛型与Class
避免强制转换
泛型参数化类型 getGenericType();
普通类型 getType()
