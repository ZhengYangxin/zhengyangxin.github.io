---
title: Android架构MVX
copyright: true
date: 2019-10-18 22:40:23
tags: [MVC, MVP, MVVM]
category: Android
---

### MVC架构设计
#### 流程关系
1. View接受用户交互事件
2. View将用户的操作，交给Controller
3. Controller完成具体业务逻辑
4. 得到封装好的Model,再进行View的更新

#### 缺点
Controller是作为媒介，处于Model和View之前，Model和View之间有紧密的联系，耦合性强
Controller做的事情过多，违反了面向对象的单一职责原则 

### MVP架构设计
#### 流程关系
1. View接受用户交互事件
2. View把用户的操作交给了Presenter
3. presenter控制Model进行业务逻辑的处理
4. presenter处理完毕后，数据封装进Model
5. presenter收到通知后，再更新数据

#### 优点
双向通信方式
1. View层与Model层完全解耦
2. 所有的逻辑都交给Presenter处理
3. MVP分层较为严谨

#### MVVM架构设计
#### 流程关系
1. View接受用户数据
2. View把用户的操作交给了ViewModel
3. ViewModel控制Model进行业务处理
4. ViewModel处理完毕后，数据封装进Model，刷新View

#### 优点
1. 降低耦合度：一个ViewModel层可以绑定不同的View层，当Model变化时View可以不变
2. 可以把一些视图逻辑放在ViewModel层中，让很多View可以重用这些视图逻辑


#### dataBinding的使用
1.gradle文件添加，支持
```
// 添加DataBinding依赖
    dataBinding{
        enabled = true
    }
```

2. 布局文件中，添加最外层layout根布局
```
<layout xmlns:android="http://schemas.android.com/apk/res/android"></layout>
```
3. 添加定义data标签，定义数据及数据来源
```
 <!-- 定义该View（布局）需要绑定的数据来源 -->
    <data>
        <variable
                name="user"
                type="com.zyx.cherish.mvp.bean.UserInfo" />
    </data>
```
4. 在正常布局中用data中定义的name去使用“@=”是为了实现双向绑定（v -> model）
```
<EditText
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@={user.name}" />
```
5. build之后在activity中通过DatabindinngUtils绑定类与布局,然后就可以设置数据
```
ActivityLoginLayoutBinding viewDataBinding = DataBindingUtil.setContentView(this, R.layout.activity_login_layout);

viewDataBinding.setUser(userInfo);
```
#### 源码分析
**内存消耗的缺点**
1. 会生成大量的object的对象，用到了数组保存数组
2. 每个View会监听数据变化，创建了线程
3. 在刷新时会轮训发送handler消息
在定义布局后通过rebuild，会生成额外的文件，这个是系统通过APT生成的
主要有
1. activity_login_layout-layout.xml 配置文件，可以让dataBing方便查找View
2. activity_login_layout 为每个View添加了Tag如binding_1
3. DataBinderMapperImpl 这个是具体实现绑定类
4. 在DataBindingUtil.setContentView(this, R.layout.activity_login_layout)后会对布局中的控件进行绑定
5. 线程 mRebindRunnable  控件会对M数据修改做监听，来更新V的数据
6. InverseBindingListener 控件自身数据修改即V数据修改，会更新M的数据
