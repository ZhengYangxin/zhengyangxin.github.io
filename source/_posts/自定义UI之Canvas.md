---
title: 自定义UI之Canvas
copyright: true
date: 2019-09-27 12:12:26
tags: [canvas]
category: "Android"
---
#### Canvas绘制几何图形，文本，位图等
1. drawBitmap() 在指定坐标绘制位图
2. drawLine()根据起始点绘制连线
3. drawPath()根据给定的路径，绘制连线
4. drawPoint()根据给定的坐标，绘制点
5. drawText()根据给定的坐标和文本绘制文字
.....

#### 位置，形状变化等
1. void translate(float x, float y) 平移操作
2. void scale() 缩放
3. void rotate() 旋转
4. void skew() 倾斜
5. void clipXXX()剪切
6. void clipOutXXX()反向剪切
7. setMatrix(Matrix matrix) 通过matrix实现以上效果

#### Canvas的状态保存
Canvas内部维护着状态栈，通过save和restore保存恢复
离屏绘制saveLayer
