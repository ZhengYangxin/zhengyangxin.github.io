---
title: 重学Java
copyright: true
date: 2020-01-08 22:08:21
tags: [Java]
author: Yangcy
top: true
img: https://images.idgesg.net/images/article/2019/05/java_binary_code_gears_programming_coding_development_by_bluebay2014_gettyimages-1040871468_2400x1600-100795798-large.3x2.jpg
coverImg: https://cn.bing.com/th?id=OHR.TrakaiLithuania_ZH-CN0447602818_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
summary: 通过阿里云大学的Java学习路线学习视频进行学习摘要总结
category: Java
---
## [阿里云Java学习路线](https://developer.aliyun.com/learning/roadmap/java?source=5176.11533457&userCode=r3yteowb&type=copy&spm=5176.12901015.0.i12901015.510b525c9IzVPf)
### Java语言基础
#### Java编程入门
##### 认识Java
1. Java的起源是SUM公司开发的，后来被oracle收购
2. Java的开发开发有三种
 * JAVA SE 桌面应用开发
 * JAVA ME 嵌入是开发 
 * JAVA EE 企业平台开发，即互联网平台开发

#### Java语言特点
1. Java是半开源的项目，可以接触底层代码
2. Java是一种编程语言，面向对象的编程思想并且一直在拓展扩充
3. 提供有方便的内存回收机制
4. 避免了复杂的指针问题，使用更加简单的引用来代替指针
5. Java是支持多线程开发的语言，使得在单位时间内的提升了处理能力
6. Java提供了高效的网络处理能力，基于NIO实现了更加高效的网络传输能力
7. Java具有良好的可移植性
8. 足够简单

#### Java的可移植性
在于同一个程序可以在不同的操作系统中执行部署，减少开发难度。依赖于Java虚拟机JVM，不同操作系统拥有不同版本的JVM，实现了移植性。
##### Java程序运行机制
* 编译型，解释型。Java是两种高级编程语言的结合，先编译成.class文件，再解释成计算机识别的机器指令
* 编译命令：Javac.exe
* 解释命令：Java.exe
* Java程序的组成：Java源文件，字节码文件，机器码指令

#### JVM
* 一台模拟的计算机，可以读取并处理经编译过的与平台无关的字节码class文件
* java编译器针对JVM产生class文件，因此独立于平台
* Java解释器负责将JVM的代码在特定的平台上运行

#### JDK的介绍
是Java的开发工具包,主要版本迭代，其中JRE是运行环境，只提供解释功能不提供程序的开发功能
* 1995.05.23，JDK1.0发布， 1996年提供对外
* 1998.12.04 JDK1.2，更名为Java2
* 2005.05.23 十周年大会，JDK1.5 ，带来了新特性，决定了未来10的核心技术
* 2014 JDK1.8，支持了Lambda，可以函数式编程
* 2017 JDK1.9,提高了1.8稳定性
* 2018 JDK1.10,属于1.9的稳定版

#### JDK的安装与配置

#### Java编程起步
创建HelloWorld.java文件，编写源文件，编写输出HelloWord的程序。Java程序是需要经过两次处理之后才能正常执行的
* 对源代码进行编译：Javac xxx/HelloWord.java，会出现一个HelloWord.class的字节码文件，利用JVM进行编译，编译出一套与平台无关的字节码文件(*.class)
* 在JVM上进行程序的解释执行Java xxx/HelloWord。解释的就是字节码文件，字节码文件的后缀是不需要编写的
* 类的定义有两种形式，public class 类名， class 类名。第一种文件名必须与类名一致，第二种可以不一致，生成的*.class文件名与类名一致。一般情况是一个class并且以public修饰，类名是驼峰式
* main方法，程序运行入口主方法

#### ClASSPATH环境属性
* PATH: 是操作系统提供的路径配置，定义所有可执行程序的路径
* CLASSPATH: 是JRE提供的，用于定义Java程序解释时类加载路径，默认是源文件的目录

#### 注释
* 单行注释 //
* 多行注释 /* .... */
* 文档注释 /** .... */

#### 标识符和关键字
* Java语言中有不同的结构：类，方法，变量结构等，对于结构的说明实际上就是标识符，是有命名规则的。
* 关键字，是系统对于一些结构的的描述处理，有着特殊含义，public static final 等等

#### Java数据类型简介
* 数据分类：基本数据类型(数字单元)分三大类，数值型，浮点型，字符型，引用数据类型(内存关系的使用)分数组，类，接口
| 类型 | 包括 | 默认值 |
| --- | --- | ---|
| 基本数据类型|||
| 整型 | byte,short,int,long| 0|
| 浮点型| float，double|0.0|
| 布尔型| boolea| false|
| 字符型| char| '\u0000'|
| 引用类型|数组，类，接口|null|
* 使用原则： 如果是描述数字首选int,double；如果要进行数据传输或者文字编码选择byte(二进制处理)；处理中文char;描述内存或者文件大小，表的主键列long
* 内存溢出：如果数值的操作超出了数值类型的范围就会陷入循环的现象，通过使用范围更大的数值类型解决

