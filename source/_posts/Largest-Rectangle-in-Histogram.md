---
title: Largest Rectangle in Histogram and Maximal Rectangle
copyright: true
date: 2017-11-28 09:52:03
tags: [leetCode, 算法]
category: "算法"
---
题目链接：
[Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/description/ "Optional title")
[Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/description/ "Optional title")
<!-- more -->
这道题算是比较难的一道题，最简单的做法就是对于任意一个bar，向左向右遍历，知道高度小于该bar。这时候计算该区域的面积。对于每一个bar，我们都做如上处理，最后得到最大值。当然这样的做法是O(n2)，过不了大数据集合测试。

从上面我们直到，对于任意一个bar n，我们得到的包含该bar n的矩形区域里面bar n是最小的。我们使用ln和rn来表示bar n向左及向右第一个小于bar n的bar的索引位置。
譬如题目中bar 2的高度是5,它的ln为1，rn为4.包含bar的矩形区域面积为（4-1-1）*5=10

我们可以从左往右遍历所有的bar，并将其push到一个stack中，如果dangqianbar的高度小于栈顶bar，我们pop出栈顶bar，同时以该bar计算举行面积。纳闷我们如何知道该bar的ln和rn呢？rn铁定就是当前遍历到的bar的索引，而ln则是当前栈顶bar的索引，因为此时栈顶bar的高度一定小于pop出来的bar的高度。

#### 复杂度分析
因为要找三个元素，所以时间复杂度为O(n)，空间复杂度为O(1)

#### 代码
```
public static int largestRectangleArea(int[] height ){
        int maxArea  = 0;
        Stack<Integer> s = new Stack<>();
        int i = 0;
        while (i <= height.length){
            int h = (i == height.length ? 0 : height[i]);
            if(s.isEmpty() ||  h >= height[s.peek()]){
                s.push(i);
                i++;
            }else {
                int t = s.pop();
                maxArea = Math.max(maxArea, height[t] * (s.isEmpty() ? i : i - s.peek() - 1));
            }
        }
        return maxArea;
    }
```