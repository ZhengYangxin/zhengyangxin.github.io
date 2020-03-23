---
title: Android 面向切面编程详解
copyright: true
date: 2019-12-27 21:38:50
author: Yangcy
img: https://cn.bing.com/th?id=OHR.TrakaiLithuania_ZH-CN0447602818_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
coverImg: https://cn.bing.com/th?id=OHR.TrakaiLithuania_ZH-CN0447602818_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
summary: 这是你自定义的文章摘要内容，如果这个属性有值，文章卡片摘要就显示这段文字，否则程序会自动截取文章的部分内容作为摘要
tags: [Android, AOP]
category: "Android"
---
## 学习目标
1. 什么是AOP（WHAT）
2. AOP的使用 (HOW)
3. 比较主流AOP方案的优缺点 (WHY)
4. 基于AOP实现的业务开源库 (WHERE)

### AOP的概念
#### 什么是AOP
* AOP与OOP一样，是一种程序设计的思想：面向切面编程(Aspect Oritented Programming)，而非技术手段。思想的实现方式是一种技术，即通过预编译方式和运行期动态代理的方式实现程序功能的统一维护
* AOP是OOP的延续，是软件开发中的热点。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各个部分之间的耦合度降低，提高程序的可重用性，提高开发效率。

#### 什么是OOP
+ OOP机面向对象编程(Object Oriented Programming)，被理解为是一种将程序分解为封装数据及相关操作的对象的编程思想。它有三大特性：多态，继承，封装。其中封装指：隐藏对象的属性和实现细节，仅对外公开访问方法，控制在程序中属性的读和写的级别，以获得更加清晰高效的逻辑单元划分。这个程序的六大设计原则中的单一职责原则一致：一个类只负责一件事。
+ 因此针对上面的封装特性，他存在一个问题：当存在关注点聚焦的场景时，他无法很好的解决，因为一个关注点是面向所有而不是一个单一的类，不受类的边界的约束,因此它只能分散到各个类,方法中去。这样的好处是降低了类的复杂性，提高了程序的可维护性，但同时他也使代码变得啰嗦了，例如添加方法的调用日志，那就必须为所有的需要日志的方法添加调用日志的方法。

#### AOP和OOP的关系
+ 面对上述聚焦某个点的问题，AOP可以理解为是针对业务处理过程中的切面进行提取，它所面对的是处理过程中的某个步骤或者阶段，以获得逻辑过程中各个部分之间低耦合性隔离效果。这两种思想在目标上有本质上的差异，但两者不是对立的，AOP是为了弥补OOP的不足。
+ OOP解决了竖向的问题，AOP则解决了横向的问题，有了AOP对程序的监控将更加简单清晰

#### 使用的业务场景
日志记录，性能统计，安全控制，事务处理，异常处理等等

#### 对比
![](/images/20171016213933903.png)
<!--more-->
现在实现上图的业务，1.为所有的方法做参数校验，2.添加前置日志，后置日志。
* 按照传统OOP实现，我们会定义一个参数校验的类Preconditions，及参数校验方法checkNotEmpty
```
public final class Preconditions {
    public static <T> T checkNotEmpty(T instance, String name) {
        if (isEmpty(instance)) {
            throw new NullPointerException(name + "不能为空");
        }
        return instance;
    }

    private static boolean isEmpty(Object obj) {
        if (obj == null) {
            return true;
        }
        if (obj instanceof String && obj.toString().length() == 0) {
            return true;
        }
        //  .......
        return false;
    }
}
```
    同样我们会定义日志记录的工具类LogDAO及写入日志方法addOpLog。这时候我们就需要找出需要需要校验参数和添加日志的方法进行一一添加。

* 而按照AOP的实现方式，是把这些横跨并嵌入众多模块的类方法集中起来，放到一个统一的地方来控制和管理，而我们只需要在这个唯一的地方进行参数的校验和日志的添加即可。

| 功能 | OOP | AOP |
| ---  | --- | --- |
|方法参数校验| 所有功能模块单独添加| 能够将同一个关注点聚焦在一处解决|
|增加日志| 所有功能模块单独添加| 能够将同一个关注点聚焦在一处解决|
|修改日志| 功能代码分散，不方便调试| 能够实现一处修改，处处生效|

### 实现方式及使用
#### 实现方式
#### APT（Annotation Processing Tool）
是一种编译器注解技术。他通过定义注解和处理器来实现编译期生成代码的功能，并且将生成的代码和源代码一起编译成.class文件。通过APT技术我们可以将横切关注点封装到注解处理器中，从而实现横向切面和业务主体的分离。
##### 在使用APT之前我们需要了解他的一些相关知识
1. Element
Element是一种在编译期间描述.java文件静态结构的一种类型，它可能表示一个package，一个class，一个filed，一个method。Element的比较应该使用equals，因为编译器间同一个Element可能会用两个对象表示.
![](/images/169dd37cf73b8b6c.png)
我们通过Element便可以获取所有需要的类结构中的所有信息，十分有用的方法：
```
public interface Element extends AnnotatedConstruct {
    //获取父Element
    Element getEnclosingElement();
    //获取子Element的集合
    List<? extends Element> getEnclosedElements();
    // 获取语言定义的类型
    TypeMirror asType()
}
```
2. TypeMirror
Element中有个asType()方法用来返回TypeMirror。TypeMirror表示java编程语言中的类型，这些类型包括基本类型，声明类型(类、接口)，数组类型，类型变量和null类型。还可以表示通配符类型参数，executable的签名和返回类型，以及对应于包和关键字的void的伪类型。我们一般用TypeMirror于类型判断。如下Activity的类型判断：
```
public interface TypeMirror extends javax.lang.model.AnnotatedConstruct {
    // 可获取获取类型，如boolean， byte，short，int等等
    TypeKind getKind();
}s
/**
 * 类型相关工具类
 */
private Types typeUtils;
/**
 * 元素相关的工具类
 */
private Elements elementUtils;
private static final String ACTIVITY_TYPE = "android.app.Activity";
private boolean isSubActivity(Element element){
    //获取当前元素的TypeMirror
    TypeMirror elementTypeMirror = element.asType();
    //通过工具类Elements获取Activity的Element，并转换为TypeMirror
    TypeMirror viewTypeMirror = elementUtils.getTypeElement(ACTIVITY_TYPE).asType();
    //用工具类typeUtils判断两者间的关系
    return typeUtils.isSubtype(elementTypeMirror,viewTypeMirror)
}
```
3. Types, typeUtil是类型相关的工具类，主要用于与TypeMirror结合使用，常用方法有
```
public interface Types {
    // 将类型转化为对应的程序元素
    Element asElement(TypeMirror t);
    // 比较类型是否相同
    boolean isSameType(TypeMirror t1, TypeMirror t2);
    // t1是否是t2子类型
    boolean isSubtype(TypeMirror t1, TypeMirror t2);
    // t1是否是t2的父类型
    boolean isAssignable(TypeMirror t1, TypeMirror t2);
    // t1是否包含t的类型，，如t1是一个类元素的typeMirror,他的内部元素包含属性，方法等
    boolean contains(TypeMirror t1, TypeMirror t2);
    // .....
}
```
4. Elements, elementUtil是元素相关的工具类，常用方法有
```
public interface Elements {
    // 通过全限定名，获取包元素
    PackageElement getPackageElement(CharSequence name);

    // 通过全限定名，获取类元素
    TypeElement getTypeElement(CharSequence name);

    //  获取包元素
    PackageElement getPackageOf(Element type);

    // 获取类元素的所有成员元素
    List<? extends Element> getAllMembers(TypeElement type);

    // ......
}
```
5. 具体实战可以参考ButterKnife、Dagger、ARouter、EventBus3、DataBinding、AndroidAnnotation，框架的实现方式可以分几步
    - 自己定义代码结构
    - 通过apt生成代码
    - 定义Manager管理器进行初始化，传入目标对象进行逻辑代码的初始化
6. demo参考,其中代码生的比较繁琐，基于字符拼接容易错，推荐使用Square的[javapoet](https://github.com/square/javapoet)库，提供了非常友好的api
```
自动生成xxx$$Proxy.java文件
public class MainActivity$$Proxy {
    public static Class<?> findTargetClass(String path) {
        if (path.equals("ddasdas")) {
            return MainActivity.class;
        }
        return null;
    }
}
//代码生成器
public class ParamaterCheckApt extends AbstractProcessor {

    // 操作Element工具类 (类、函数、属性都是Element)
    private Elements elementUtils;

    // type(类信息)工具类，包含用于操作TypeMirror的工具方法
    private Types typeUtils;

    // Messager用来报告错误，警告和其他提示信息
    private Messager messager;

    // 文件生成器 类/资源，Filter用来创建新的源文件，class文件以及辅助文件
    private Filer filer;

    // 该方法主要用于一些初始化的操作，通过该方法的参数ProcessingEnvironment可以获取一些列有用的工具类
    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
        S// ....
    }

    /**
     * 相当于main函数，开始处理注解
     * 注解处理器的核心方法，处理具体的注解，生成Java文件
     * @param annotations 使用了支持处理注解的节点集合
     * @param roundEnv 当前或是之前的运行环境,可以通过该对象查找找到的注解。
     * @return true 表示后续处理器不会再处理（已经处理完成）
     */
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        if (annotations.isEmpty()) {
            return false;
        }
        Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(Arouter.class);
        for (Element element : elements) {
            // 通过类节点获取包节点
            String packageName = elementUtils.getPackageOf(element).getQualifiedName().toString();
            // 获取简单类名
            String className = element.getSimpleName().toString();
            messager.printMessage(Diagnostic.Kind.NOTE, "被注解的类有：" + className);
            String finalClassName = className + "$$Proxy";
            try {
                JavaFileObject sourceFile = filer.createSourceFile(packageName + "." + finalClassName);
                // 定义Writer对象，开启写入
                Writer writer = sourceFile.openWriter();
                // 设置包名
                writer.write("package " + packageName + ";\n");

                writer.write("public class " + finalClassName + " {\n");

                writer.write("public static Class<?> findTargetClass(String path) {\n");

                // 获取类之上@ARouter注解的path值
                Arouter aRouter = element.getAnnotation(Arouter.class);

                writer.write("if (path.equals(\"" + aRouter.path() + "\")) {\n");

                writer.write("return " + className + ".class;\n}\n");

                writer.write("return null;\n");

                writer.write("}\n}");

                // 最后结束别忘了
                writer.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return false;
    }
}
```

7. 生成的代码可以在project/项目(app)/build/generated/source/apt/ 下面可以找到

#### AspectJ
* 是一种编译器，它在Java编译器的基础上增加了关键字的识别和编译方法，因此AspectJ可以织入Java代码。他还提供了AspectJ程序，在编译期间将开发者编写的AspectJ程序织入到目标程序中。它的核心是ajc(aspectjtools编译器)和weaver(织入器aspectjweaver)。
* aspectjtools编译器是基于java编译器之上的，可以编译.aj文件，在java编译器之上加了关键字和方法，因此也可以编译java代码
* weaver织入器：为了在java编译器上使用AspectJ而不依赖于AJC编译器，AspectJ5出现了@AspectJ，使用注解的方式去编写AspectJ代码，可以在任何java编译器上使用。在代码编译期间扫描目标程序，根据切点(PointCut)匹配，将开发者编写的Aspect程序编织到目标程序的.class文件中，对目标程序作了重构（重构的单位是JoinPoint），目的就是建立目标程序的与Aspect程序的连接（获取执行对象，方法，参数等上下文），从而达到Aop的目的

##### Aspect的一些术语
1. 切面(Aspectj): 即一个关注点的模块化，这个关注点可能横跨多个对象，其实就是公共功能的实现。如日志切面，权限切面，事物切面等。

2. 通知（Advice）：是切面的具体实现。以目标方法为参照点，根据放置的位置不同，可以分为：
    * 前置通知(before)
    * 后置通知(after)
    * 异常通知(AfterThrowing)
    * 最终通知(AfterReturning)
    * 环绕通知(Around)

3. 在实际应用中通常是切面类中的一个方法，具体哪个则取决于配置。
    * 切入点（PointCut）： 用于定义通知应该切入到那些连接点上。不同的通知通常需要切入到不同的连接点上，这种精准的匹配依赖于切入点的正则表达式定义。连接点JointPoint：就是程序在运行过程中能够切入到切面的地点。列如： 方法调用，异常抛出修改字段等。
    * 目标对象（Target Object）：包含连接点的对象，也被称作被通知或者被代理的对象，这些对象只剩下干干净净的核心业务逻辑代码，所有共有功能的代码等则是等待Aop的切入
    * AOP代理（AOP Proxy）：将通知应用到目标对象之后动态的创建对象。代理对象的功能等于目标对象的核心业务逻辑功能加上共有功能
    * 织入(Weaving): 将切面应用到目标对象从而创建一个新的代理对象的过程，这个过程可以发生在编译期，类装载期及运行期，不同的时期有着不同的条件。如AspectJ则需要一种支持AOP的特殊编译器；发生在类装载期，就要求有一个支持AOP实现的特殊类装载器；发生在运行期，则可直接通过java语言的反射机制与动态代理机制来实现

4. AspectJ中的Join Point

| Join Points | 说明 | 实例 |
| ---  | --- | --- |
| method call | 函数调用| 比如Log.e()调用的地方是一个joinPoint |
| method execution| 函数执行| 比如Log.e()的内部执行，是一出joinPoint|
| constructor call| 构造函数的调用|s 和method call 类似|
| constructor execution| 构造函数的执行| 和method execution类似|
| field get | 获取某个变量| 比如读取DemoActivity.debug成员|
| field set | 设置某变量| 比如设置DemoActivity.debug变量|
| preinitialization| Object 在构造函数中做一些工作| 很少使用|
| initialization| Object在构造函数中做得工作| 很少使用|
| static initialization | 类初始化 | 比如类的static{} |
| handler | 异常处理 | 比如try catch（xxx）中，对应catch内的执行|
| advice execution | AspectJ的内容| ... |


5. PointCut基于正则表达式@注解 访问权限 返回值的类型 包名.函数名(参数),call(public  *  *.println(..)) 
是匹配一个方法，第一个*表示方法的返回值，第二个*表示方法的包名，(..)表示方法的参数的样子，..代表任意个数，任意类型的参数
* *表示任何数量的字符，除了(.)
* ..表示任何数量的字符包括任何数量的(.)
* +描述指定类型的任何子类或者子接口
* 同java一样，提供了一元和二元的条件表达操作符。
s一元操作符：!
二元操作符：||和&&
* 参考[深入理解Android之Aop](https://blog.csdn.net/innost/article/details/49387395)

