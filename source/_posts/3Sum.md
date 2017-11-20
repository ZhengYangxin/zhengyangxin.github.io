---
title: 3Sum
date: 2017-11-20 15:03:24
tags: [leetCode, 算法]
category: "算法"
copyright:
---
题目链接：[3Sum](https://leetcode.com/problems/3sum/discuss/ "Optional title")
Given an array S of n integers, are there elements a, b, c in S such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.
Note: The solution set must not contain duplicate triplets.
```
For example, given array S = [-1, 0, 1, 2, -1, -4],
A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```
<!-- more -->
#### 题目的两点要求：
1. 每个答案组里面的三个数字是要从小到大排列起来的
2. 每个答案不可以和其他的答案相同

#### 题目分析：
1. 每个答案数组triplet中的元素是要求升序排列的
2. 不能包含重复的答案数组

#### 解题思路：
1. 因为要求每个答案数组中的元素都是升序排列的，所以开头我们要对数组进行排序
2. 因为不能包含重复的答案数组，我们要在代码里做去重操作
3. 归根结底是Two pointers的想法，定位其中两个指针，根据和的大小移动另外一个

#### 复杂度分析
因为要找三个元素，所以时间复杂度为O(n2)，空间复杂度为O(1)

#### 代码
```
public static List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    if(nums.length < 3)
        return result;
    Arrays.sort(nums);
    int i = 0;
    while (i < nums.length -2){
        if(nums[i] > 0)
            return result;
        int j = i + 1;
        int k = nums.length - 1;

        while (j < k){
            int sum = nums[i] + nums[j] + nums[k];
            if(sum == 0)
                result.add(Arrays.asList(nums[i], nums[j], nums[k]));
            if(sum <= 0)
                while (nums[j] == nums[++j] && j < k);
            if(sum >= 0)
                while (nums[k] == nums[--k] && j < k);
        }
        while (nums[i] == nums[++i] && i < nums.length - 2);
    }
    return result;
}
```