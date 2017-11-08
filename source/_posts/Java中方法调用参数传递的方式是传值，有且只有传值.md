---
title: Java中方法调用参数传递的方式是传值，有且只有传值
date: 2017-06-26 22:56:10
tags: [Java基础]
---
#### Java中方法调用参数传递的方式是传值，尽管传的是引用的值而不是对象的值。（Does Java pass by refrence or pass by value）

**基本数据类型与引用数据类型**

- 8种基本数据类型

- 引用数据类型：类、接口类型、数组类型、枚举类型、注解类型。

**区别：**

- 基本数据类型在被创建时，在栈上给其划分一块内存，将数值直接存储在栈上。

- 引用数据类型在被创建时，首先要在栈上给其引用（句柄）分配一块内存，而对象的具体信息都存储在堆内存上，然后由栈上面的引用指向堆中对象的地址。

例如，有一个类Person,有属性name,age,带有参的构造方法，

Person p = new Person("zhangsan",20);

**在内存中的具体创建过程是：**

1.首先在栈内存中位其p分配一块空间;

2.在堆内存中为Person对象分配一块空间，并为其三个属性设初值""，0；

3.根据类Person中对属性的定义，为该对象的两个属性进行赋值操作；

4.调用构造方法，为两个属性赋值为"Tom",20；（注意这个时候p与Person对象之间还没有建立联系）；

5.将Person对象在堆内存中的地址，赋值给栈中的p;通过引用（句柄）p可以找到堆中对象的具体信息。

#### 相关知识：

静态区： 保存自动全局变量和 static 变量（包括 static 全局和局部变量）。静态区的内容在整个程序的生命周期内都存在，由编译器在编译的时候分配。

堆区：  一般由程序员分配释放，由 malloc 系列函数或 new 操作符分配的内存，其生命周期由 free 或 delete 决定。在没有释放之前一直存在，直到程序结束，由OS释放。其特点是使用灵活，空间比较大，但容易出错

栈区： 由编译器自动分配释放，保存局部变量，栈上的内容只在函数的范围内存在，当函数运行结束，这些内容也会自动被销毁，其特点是效率高，但空间大小有限

文字常量区： 常量字符串就是放在这里的。   程序结束后由系统释放。

程序代码区：存放函数体的二进制代码。

![Alt text](http://images0.cnblogs.com/blog2015/751291/201507/051423530158394.png "Optional title")

**在Java中，所有的对象变量都是引用，Java通过引用来管理对象。然而在给方法传参时，Java并没有使用传引用的方式，而是采用了传值的方式。**
例如下面的badSwap()方法：
```
public void badSwap(int var1, int var2)  
{  
  int temp = var1;  
  var1 = var2;  
  var2 = temp;  
}  
```
当badSwap方法结束时，原有的var1和var2的值并不会发生变化。即使我们用其它Object类型来替代int，也不会有变化，因为Java在传递引用时也是采用传值的方式。（译者注：这里是关键，全文的核心是：1. Java中对象变量是引用 2. Java中方法是传值的 3. 传方法中参数时，传递的是引用的值）
代码：

```
public void tricky(Point arg1, Point arg2)  
{  
  arg1.x = 100;  
  arg1.y = 100;  
  Point temp = arg1;  
  arg1 = arg2;  
  arg2 = temp;  
}  
public static void main(String [] args)  
{  
  Point pnt1 = new Point(0,0);  
  Point pnt2 = new Point(0,0);  
  System.out.println("X: " + pnt1.x + " Y: " +pnt1.y);   
  System.out.println("X: " + pnt2.x + " Y: " +pnt2.y);  
  System.out.println(" ");  
  tricky(pnt1,pnt2);  
  System.out.println("X: " + pnt1.x + " Y:" + pnt1.y);   
  System.out.println("X: " + pnt2.x + " Y: " +pnt2.y);    
}  
```
执行main()的输出如下：

```
X: 0 Y: 0  
X: 0 Y: 0  
X: 100 Y: 100  
X: 0 Y: 0  
```
这个方法成功地改变了pnt1的值，但pnt1和pnt2的交换却失败了！这是Java参数传递机制里最让人迷惑的地方。在main()中，pnt1和pnt2是Point对象的引用，当将pnt1和pnt2传递给tricky()时，Java使用的正是传值的方式，将这两个引用的传给了﻿﻿arg1和arg2。也就是说arg1和arg2正是pnt1和pnt2的复制，他们所指向的对象是相同的。详情可见下面的图示：
<div align=center>
![Alt text](http://images.techhive.com/images/idge/imported/article/jvw/2000/05/03-qa-0512-pass1-100158781-orig.gif "Optional title")
在作为参数传递后，对象至少有两个引用指向自己
</div>
在main()中，引用被复制并以传值的方式进行传递，对象本身并不会被传递。因此，tricky()方法中pnt1所指向的对象发生了变化。因为传递的是引用的复制，因此引用的交换既不能引起对象的交换，更不会使原始引用发生变化。如图2所示，tricky()交换了arg1与arg2，但不会影响pnt1和pnt2。因此若想交换原始引用pnt1和pnt2，那么不能通过调用方法的方式来实现。
<div align=center>
![Alt text](http://hi.csdn.net/attachment/201201/31/0_13279907114A2K.gif "Optional title")
在作为参数传递后，对象至少有两个引用指向自己
</div>

#### 总结：####
1. Java中对象变量是引用 
2. Java中方法是传值的 
3. 传方法中参数时，传递的是引用的值 



