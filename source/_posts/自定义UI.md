---
title: 自定义UI之Paint
copyright: true
date: 2019-09-26 12:24:40
tags: [自定义UI, shader, colorFilter, xfermode]
category: Android
---
#### Paint画笔
画笔的实现内部都是调用native方法的，画笔的基础Api
1. setColor(Color.RED)
2. setARGB(),setAlpha  设置颜色及透明度
3. setAntiAlias(true) 抗锯齿
4. setStyle() Paint.Style.STROKE/FILL/FILL_AND_STROKE  描边，填充效果
5. setStrokeWidth() 描边宽度
6. setStrokeCap() Paint.Cap.BUTT/ROUND/SQUARE  线尾部形状 默认BUTT,round圆角效果，方形
7. setStrokeJoin() 拐角效果，MITER/ROUND/BEVEL 默认MITER尖角，圆角，折角
8. setShader(Shader shader) 设置环形渲染器
9. setXfermode() 设置图层混合模式
10. setColorFilter()设置颜色过滤器
11. setFilterBitmap(true) 设置双线性过滤，使图片更加平滑
12. setMaskFilter() 设置画笔的遮罩滤镜
13. setTextScalX(2) 设置缩放文本倍数
14. setTextSize(38) 设置文本字体大小
15. setTextAlign() 对齐方式
16. setUnderLineText(true) 设置下划线
17. getTextBounds(str, 0, str.length(), rect) 测量文本大小，将文本大小信息存放在rect中
18. measureText(str) 测量文本的宽
19. getFontMetrics()获取字体度量对象, 变量包括ascent,descent,top,bottom,leading ,字体的高度为descent - ascennt, leading为行间距是当前字的ascent - 上一个字的descent

#### setShader(Shader shader) 设置环形渲染器(着色器)
1. LinearGradient 线性渲染
2. RadilGradient 环形渲染
3. SweepGradient 扫描渲染
4. BitmapGradient 位图渲染

##### TileMode 平铺模式
1. CLAMP 超出后 以最后像素拉升填充  
2. REPEAT 超出后重复平铺
3. MIRROR 镜像填充

#### 组合渲染器，图层混合模式Xfermode   
**PortorDuff.Mode**
1. 使用场景
    * ComposeShader 组合渲染器
    * Paint.setXfermode()
    * porterDuffColorFilter 颜色过滤器
2. 使用注意
    * 静止硬件加速，14之后图层混合不支持硬件加速，但系统是默认打开的setLayerType(View.LAYER_SOFTWARE, null)
    *  离屏绘制，去除背景等其他干扰,保证xfermode不会出现错误结果saveLayer  ,restoreToCount,
#### setColorFilter
**LightColorFilter**
颜色过滤器，可以过滤掉指定颜色
**PorterDuffFilter**
颜色和其他的图层混合
**颜色矩阵ColorMatrixColorFilter**
通过修改颜色矩阵Matrix数组中的值

#### 画笔滤镜总结
1. 简单的模拟光照效果，可以使用LightColorFilter
2. 图像与颜色的，图层混合的实现，PorterDuffColorFilter
3. 颜色数组及颜色举证实现滤镜效果


