---
title: Find Mininum in Rotated Sorted Array I and II
copyright: true
date: 2017-11-24 10:04:15
tags: [leetCode, 算法]
category: 算法
---
题目链接:
[Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/ "Optional title")
[Find Minimum in Rotated Sorted Array II](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/description/ "Optional title")

Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.
(i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2).
Find the minimum element.
**I和II的区别在于是否有重复元素**
<!-- more -->

#### 题目的要求
1.  一个升序的数组且无重复元素
2.  在处理数组前进行部分反转
3.  找出最小元素

#### 题目分析
1. 寻找最小值，我们可以用二分查找法来做
2. 在一个区间的A，如果A[start] < A[end]，那么该区间一定是有序的

#### 解题思路
假设一个轮转的排序数组arr，我们首先获取中间元素的值，arr[mid], mid = start + (end - start)/2.因为没有重复数组，那么就有两种情况。
1. arr[mid] > arr[start], 那么最小值一定在右半区间，eg:{4,5,6,7,0,1,2} 中间数为7.7>4,最小元素一定在{7,0,1,2}
2. arr[mid] < arr[start], 那么最小值一定在左半区间,eg:{7,0,1,2,3,4,5,6}中间数为2,2<7，最小元素一定在{7,0,1,2}
3. 处理重复元素，当arr[mid] == arr[start], 跳过start++.

#### 边界值
1. 输入数组arr == null, arr.length == 0 ,return 0
2. arr.length == 1, return arr[0]
3. 输入数组arr为有序数组，return arr[start];

#### 复杂度
时间复杂度为O(logn)，空间复杂度为O(N)
 
#### 代码
```
public static int findMin(int[] arr){
    if(arr == null || arr.length == 0)
        return 0;
    if(arr.length == 1)
        return arr[0];

    int start = 0;
    int end = arr.length -1;

    while (start < end){
        if(arr[start] < arr[end]){
            return arr[start];
        }
        int min = start + (end - start)/2; It can avoid overflow.
        if(arr[start] < arr[min]){
            start = min;
        }else if(arr[start] < arr[min]){
            end = min;
        }else{
            start++; //处理重复元素
        }
    }
    return arr[start] < arr[end] ? arr[start] : arr[end];
}
```

