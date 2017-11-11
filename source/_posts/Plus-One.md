---
title: Plus One
date: 2017-11-11 10:35:02
tags: [leetCode,算法]
category: "算法"
---
题目链接 [Plus One](https://leetcode.com/problems/plus-one/description/ "Optional title")
Given a non-negative number represented as an array of digits, plus one to the number.
The digits are stored such that the most significant digit is at the head of the list.
 这道题很简单，就是考加法的进位运算问题对一个数组进行加一操作，代码如下：
 ```
 //
 public static int[] plusOne(int[] digit){
        int one = 1;
        int sum = 0;
        for(int i = digit.length -1 ; i >= 0; i--){
            sum = digit[i] + one;
            digit[i] = sum % 10;  //取余
            one = sum / 10;  //取整
            if(one == 0){ //无需进位，则返回即可
                return digit;
            }
        }
        int[] res = new int[digit.length + 1]; //当最高位仍需进位则重新new一个数组
        res[0] = 1;
        return res;
    }
 ```
 具体思路是维护一个进位，对每一位进行加一，然后判断进位，如果有继续到下一位，否则就可以返回了，因为前面不需要计算了。有一个小细节就是如果到了最高位进位仍然存在，那么我们必须重新new一个数组，然后把第一个为赋成1（因为只是加一操作，其余位一定是0，否则不会进最高位）。因为只需要一次扫描，所以算法复杂度是O(n)，n是数组的长度。而空间上，一般情况是O(1)，但是如果数是全9，那么是最坏情况，需要O(n)的额外空间。