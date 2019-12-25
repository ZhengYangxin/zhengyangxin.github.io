---
title: Android事件分发源码分析
copyright: true
date: 2019-09-29 13:18:15
tags: [源码分析及架构]
category: "Android"
---
### 源码分析
#### 事件分发的类型
down，move， up，cancel
#### 事件分发序列
down -> up
down -> move -> up
#### 事件分发的对象
Activity  ViewGroup  View
#### 事件分发的方法
dispatchTouchEvent()
onInterceptTouchEvent()
onTouchEvent()

#### Activity的事件分发
1. dispatchTouchEvent
2. PhoneWindow.superDispatchTouchEvent()
3. DecorView.superDispatchTouchEvent
4. ViewGroup.dispatchTouchEvent
事件由Activity到了ViewGroup，如果View没有消费掉，会调用Activity的onTouchEvent

#### ViewGroup的事件分发
1. 事件从Down开始，接收到Down事件时会做取消并清除touch事件，置空mFistTouchTarget对象，清空Touch标记位
2. 是否拦截判断(除Down时间以外），通过onIntercept，子类可以通过修改requestDisallowInteceptTouchEvent使父容器不拦截事件.默认都会走onInterceptTouchEvent方法，默认返回false
3. 会调用dispatchTranseformdTouchEvent方法,内部会调用子类的dispatchTouchEvent方法或者父类的dispatchTouchEvent的方法

#### View中的事件处理
1. 事件传递到中，会先发判断TouchListener,如果设置了TouchListener,根据回调方法onTouch方法的返回值，做响应处理，若返回true则进行第二步，否则第三步
2. onTouch中返回true，则意味着事件被消费，onTouchEvent的方法将不再执行，事件结束
3. onTouch中返回false,则意味着事件没有消费，将继续执行onTouchEvent方法
4. onTouchEvent会更具事件类型执行不同的逻辑
5. 当事件时down时，会判断是否设置了longClick事件，设置了则会更具到up的时长和longClick方法的返回值去决定是否执行onClickListener方法，修改OnLongClickListener方法的返回值可以使两个方法都执行

### 设计思想
#### 架构设计思想的需求点
1. 每个界面上的元素都需要有事件响应，那意味着事件的核心处理类不能太多
2. 事件传递的工作不能丢给开发者，只暴露接口
3. 每个控件都可以接受事件和消费事件，如果控件消费了，你那后续事件就交给这个事件了
4. 如何查找会消费事件的控件
5. 如何更具坐标查找该控件的范围
6. 每次发生事件，都需要遍历每个子元素吗？

##### 为什么所有的控件都继承字View
View是事件处理和传递的抽象类，还包括绘制，测量其他代码

#### 事件分发的思考
1. 容器包含很多子控件
2. 子控件和容器都能消费事件
3. 子控件不能进行分发给其他子控件
4. 分发功能只能由容器分发给子控件

#### 如何快速查找能响应的控件
TouchTarget消费对象的缓存队列的设计，利用链表及缓存池，通过链表操作达到复用的效果
obtain()方法，内部操作加锁，判断静态变量recycleBin缓存对象TouchTarget是否为空，不为空则取出第一个，否则new出一个TouchTarget.当一系列事件消费完后 ，释放这个对象加入缓存


