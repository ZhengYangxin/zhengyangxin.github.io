---
title: Android屏幕适配
copyright: true
date: 2019-09-30 14:33:56
tags: [屏幕适配]
category: “Android”
---
### Android的碎片化
android的碎片化严重，屏幕尺寸不同，分辨率密度，决定了做适配的重要性
#### 常见方式
1. 避免写死控件，使用wrap_content,match_parent
2. LinearLayout线性布局：权重
3. RelativeLayout绝对布局
4. ContrainLayout 约束布局
5. Percent-support-lib 百分比布局

#### 图片资源释放
1. .9图片或者SVG实现缩放
2. 备用位图匹配不同的分辨率

#### 用户流程匹配
1. 根据业务逻辑不同执行不同的跳转
2. 根据别名展示不同的界面

#### 限定符适配
1. 分辨率
2. 尺寸
3. 最小屏
4. 横竖屏

#### 刘海屏的适配
1. Android 9.0官方适配
2. 华为，OPPO，vivo 

### 适配方式
#### 限定符适配
**优点**
1. 使用简单，无需开发者手动指定
2. Google推荐使用方式，有系统自行判断
3. 适配通过不同的xml实现，无需代码加逻辑
**缺点**
1. 增加APK大小，适配机型越多，xml越多
2. 适配所以，大概增加近3m
3. 不能适配奇葩机型，如手表
#### 百分比适配
**优点**
1. 通过百分比定义宽高，比较方便
2. 彻底抛弃px,dp
3. 开发量小
**缺点**
1. 自定义不支持
2. 对代码侵入强
#### 修改系统density, densityScale, densityDpi 
#### 代码动态适配


### 网易云音乐歌单页面
#### Toolbar的使用
1. 4.4之前是使用ActionBar，之后推荐使用Toolbar
2. Toolbar不符合设计要求，通过反射修改子元素的属性(left,pading等)
#### 沉浸式设计
1. 沉浸式与非沉浸式的区别是状态栏是否透明，是否与App主题是否一致.
通过属性，开启沉浸式后状态栏就会变成浮动的，导致Toolbar向上移动
```
        <item name="android:windowTranslucentStatus">true</item>
        <item name="android:windowTranslucentNavigation">true</item>
```
通过代码控制，同样会导致Toolbar向上移动，内容延伸进状态栏
2. 通过自定义设置一个透明的statusBarView，用于定义statusBar的颜色
3. 