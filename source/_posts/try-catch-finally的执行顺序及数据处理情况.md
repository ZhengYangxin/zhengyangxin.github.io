---
title: try catch finally的执行顺序及数据处理情况
date: 2017-06-25 21:19:38
tags: [Java基础]
---
#### 结论

1. 不管有木有出现异常，finally块中的代码都会执行
2. 当try或catch中有return时，finally仍然会执行
3. 当try或catch中有return时,finally是在return前执行的（此时并没有返回运算后的结果，而是先把运算结果保存起来，而后再去执行finally，此如果finally若有rentrn，则程序结束，若无则操作后跳回try或catch继续执行）
4. finally中最好不要包含return，否则程序会提前推出，返回值不是catch或catch的值
 
#### 注意
- 例1
```
try{} catch(){}finally{} return;
显然程序按顺序执行。
```
- 例2
```
try{ return; }catch(){} finally{} return;
程序执行try块中return之前（包括return语句中的表达式运算）代码；
再执行finally块，最后执行try中return;
finally块之后的语句return，因为程序在try中已经return所以不再执行。
```
- 例3
```
try{ } catch(){return;} finally{} return;
程序先执行try，如果遇到异常执行catch块，
有异常：则执行catch中return之前（包括return语句中的表达式运算）代码，再执行finally语句中全部代码，
最后执行catch块中return. finally之后也就是4处的代码不再执行。
无异常：执行完try再finally再return.
```
- 例4
```
try{ return; }catch(){} finally{return;}
程序执行try块中return之前（包括return语句中的表达式运算）代码；
再执行finally块，因为finally块中有return所以提前退出。
```
- 例5
```
try{} catch(){return;}finally{return;}
程序执行catch块中return之前（包括return语句中的表达式运算）代码；
再执行finally块，因为finally块中有return所以提前退出。
```
- 例6
```
try{ return;}catch(){return;} finally{return;}
程序执行try块中return之前（包括return语句中的表达式运算）代码；
有异常：执行catch块中return之前（包括return语句中的表达式运算）代码；
则再执行finally块，因为finally块中有return所以提前退出。
无异常：则再执行finally块，因为finally块中有return所以提前退出。
```
#### finally块中改变返回值的特殊情况####

这里涉及到java的引用传递和值传递，首先要明白java全部都是传值的。对于基本数据类型的值是存在栈中的，它是将数据完完整整的拷贝，生成一个新的变量值；对于引用类型的数据既对象它的数据存在于堆中，栈中保存的是指向数据的引用地址，它是将对象的引用地址值拷贝了，所以修改了对象内容，但地址值是不变的。
测试代码code1
```
public class FinallyTest {
    public static void main(String[] args) {

        System.out.println(new FinallyTest().test());;
    }

    static int test()
    {
        int x = 1;
        try
        {
            x++;
            return x;
        }
        finally
        {
            ++x;
        }
    }
}
```
输出2
测试代码code2

```

public class FinallyTest2  
{
    public static void Main(string[] args)  
       {  
         /*测试test1*/   
           List<string>relist=test1();  
           foreach (var item in relist)  
           {  
               Console.WriteLine(item);  
           }  
           Console.ReadLine();  
       }  
       private static List<string> test1()  
       {  
           List<string> strlist = new List<string>();  
           strlist.Add("zs");  
           strlist.Add("ls");  
           strlist.Add("ww");  
           strlist.Add("mz");  
           try  
           {  
               strlist.Add("wq");  
               return strlist;  
           }  
           finally  
           {  
               strlist.Add("yyy");  
           }  
       } 
}
```

测试输出结果：
zs
ls
ww
mz
wq
yyy


**所以可得**
如果finally中没有return语句，但是改变了要返回的值，这里有点类似与引用传递和值传递的区别，分以下两种情况：

1）如果return的数据是基本数据类型或文本字符串，则在finally中对该基本数据的改变不起作用，try中的return语句依然会返回进入finally块之前保留的值。

2）如果return的数据是引用数据类型，而在finally中对该引用数据类型的属性值的改变起作用，try中的return语句返回的就是在finally中改变后的该属性的值。

**最终结论：**
任何执行try或者catch中的return语句之前，都会先执行finally语句，如果finally存在的话。如果finally中有return语句，那么程序就return了，所以finally中的return是一定会被return的，编译器把finally中的return实现为一个warning。
