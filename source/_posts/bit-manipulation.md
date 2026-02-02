---
title: Bit Manipulation
copyright: true
date: 2017-12-04 15:41:37
tags: [leetCode, "算法"]
category: 算法
---
### Bit Manipulation(位运算)：
一共五种运算：与，或，异或，左移，右移。
#### 常用技巧：
（1） n & （n-1）能够消灭n中最低位中的1。
（2） 右移：除以2， 左移：乘以2。
（3） 异或性质：交换律，0^a=a, a^a=0;
（3） 将常用字符、数字等均转为按位运算，可以节约空间。
<!-- more -->
### 例题
#### [Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/description/ "Optional title")
使用右移。
使用n&(n-1)可以消灭一个1的性质来求解。
```
public static int hammingWeight(int n) {
    int cnt = 0;
    while (n > 0) {
        cnt += (n & 1);
        n >>= 1;
    }
    return cnt;
}
```
#### [Missing Number](https://leetcode.com/problems/missing-number/description/ "Optional title")
这道题给我们n个数字，是0到n之间的数但是有一个数字去掉了，让我们寻找这个数字，要求线性的时间复杂度和常数级的空间复杂度。
**解法一:**最直观的一个方法是用等差数列的求和公式求出0到n之间所有的数字之和，然后再遍历数组算出给定数字的累积和，然后做减法，差值就是丢失的那个数字
```
public static int missingNumber(int[] nums) {
    int sum = 0;
    int n = nums.length;
    for(int i =0; i< n; i++){
        sum += nums[i];
    }
    return (int)(0.5 * n * (n + 1) - sum);
}
```
**解法二：**，使用位操作Bit Manipulation来解的，用到了异或操作的特性。思路是既然0到n之间少了一个数，我们将这个少了一个数的数组合0到n之间完整的数组异或一下，那么相同的数字都变为0了，剩下的就是少了的那个数字了
```
public static int missingNumber1(int[] nums) {
    int res = 0;
    for (int i = 0; i < nums.length; ++i) {
        int midres = (i + 1) ^ nums[i];
        res ^= midres;
    }
    return res;
}
```
**解法三：**这道题还可以用二分查找法来做，我们首先要对数组排序，然后我们用二分查找法算出中间元素的下标，然后用元素值和下标值之间做对比，如果元素值大于下标值，则说明缺失的数字在左边，作为读者的你可能会提出，排序的时间复杂度都不止O(n)，何必要多此一举用二分查找，还不如用上面两种方法呢。对，你说的没错，但是在面试的时候，有可能人家给你的数组就是排好序的，那么此时用二分查找法肯定要优于上面两种方法，所以这种方法最好也要掌握以下
```
public static int missingNumber2(int[] nums) {
    if(nums == null || nums.length ==0)
        return -1;
    int low = 0;
    int high = nums.length - 1;
    while (low <= high){
        int mid = low + (high - low)/2;
        if(nums[mid] > mid){
            high = mid -1;
        }else {
            low = mid + 1;
        }
    }
    return low;
}
```
#### [Power of Two](https://leetcode.com/problems/power-of-two/description/ "Optional title")
使用n&(n-1）=0来判断。
注意0和负数的情况。
这道题让我们判断一个数是否为2的次方数，而且要求时间和空间复杂度都为常数
**解法一：**那么我们很容易看出来2的次方数都只有一个1，剩下的都是0，所以我们的解题思路就有了，我们只要每次判断最低位是否为1，然后向右移位，最后统计1的个数即可判断是否是2的次方数
```
public static boolean isPowerOfTwo(int n) {
    int cnt = 0;
    while (n > 0) {
        cnt += (n & 1);
        n >>= 1;
    }
    return cnt == 1;
}
```
**解法二：**这道题还有一个技巧，如果一个数是2的次方数的话，根据上面分析，那么它的二进数必然是最高位为1，其它都为0，那么如果此时我们减1的话，则最高位会降一位，其余为0的位现在都为变为1，那么我们把两数相与，就会得到0，用这个性质也能来解题。
```
public static boolean isPowerOfTwo2(int n){
    int result = n & (n - 1);
    if(n > 0 && result == 0){
        return true;
    }else {
        return false;
    }
}
```