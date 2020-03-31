---
title: 跟我学：LeetCode刷题之1.两数之和
copyright: true
date: 2020-04-01 05:56:50
tags: [Leetcode，两数之和]
author: Yangcy
summary: 两数之和、多解法分析
category: Algorithm
---

### 导读

该题是leetcode的第一题，也是简单题。但无论题目简单难否，答案却是各式各样。有些题解代码精简明了，有些却是又臭又长；有些题解另辟蹊径，有些却是奇技淫巧，前者代码是需要我们学习，而后者是我们需要知道然后避免的。在本文我将为大家讲解该题前者代码的几种解法，后者代码解法大家充分理解本题后可以去官网逛逛评论或者题解，没准能博君一笑。这里通过一张思维导图帮助大家整理思路。思维导图一方面罗列了本文的大纲，另一方面也是希望读者朋友可以保存图片至自己的复习资料里。以便随时通过该大纲去复习。



![img](http://pb3.pstatp.com/large/pgc-image/fe2040e1de044c21aa76abf10e084544)

#### 看题总结

给定一个**整数数组 nums** 和一个目标值 target，请你在该数组中找出和**为目标值的那 两个 整数**，并返回他们的**数组下标**。

**你可以假设每种输入只会对应一个答案**。但是，你**不能重复利用**这个数组中同样的元素。

**示例:**

```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

Related Topics

数组

哈希表

通过对题目的阅读，我们可以得出以下几个结论

1. 题目类型是数组类型
2. 题意在数组中找到和为目标值的两个元素，并返回数组下标。例如num[i] + nums[j] = target。
3. 每种输入都会有对应的一个答案，意味着输入数组nums肯定是有效的，输入是合法的
4. 不能重复利用数组中的元素，即满足条件的量元素数组下标不同 i != j，输出要排除i==j的答案

### 多解法及复杂度分析

#### 解法一：暴力法：双重循环

从题意分析得出的结论,我们只需要在数组中找到下标i，j两个元素。最简单直接的方法就是通过双重循坏，第一重循环确定元素nums[i], 第二重循坏在[i+1, nums.length)中找num[j], 符合条件为nums[j] = target - nums[i];**别忘了i ！= j因为j的最大值是nums.length-1, 所以i的最大值为num.length-2**

针对下面的模板代码**墙裂建议**理解背诵默写，能通过双重循环暴力法解决的题目都可以套用

```
  for (int i = 0; i < nums.length - 1; i++) {
    for (int j = i + 1; j < nums.length; j++) {
      // 条件
      .....
    }
  }
```

完整代码如下

```
public int[] twoSum(int[] nums, int target) {
  if (nums == null || nums.length < 2) {
    return new int[0];
  }

  for (int i = 0; i < nums.length - 1; i++) {
    for (int j = i + 1; j < nums.length; j++) {
      int num = target - nums[i];
      if (num == nums[j]) {
        return new int[]{i, nums[j]};
      }
    }
  }

  return new int[0];
}
```

**复杂度分析：**

时间复杂度为O(n^2)：对于每个元素，我们试图通过遍历数组的其余部分来寻找它所对应的目标元素，这将耗费 O(n)O(n) 的时间。

空间复杂度为O(1)：没有使用辅助空间

------

#### 解法二：哈希表 + 双指针

针对这种方法只要哈希表和双指针各自的作用

1. 哈希表，该题通过Map<Integer, List<Integer>> map存储key为元素值，value为元素值相同的数组下标的集合。
2. 双指针，针对**有序数组**，定义左右头尾指针left = 0，right = nums.length - 1。通过条件nums[left] + nums[right] 与 target的比较大小，因为是有序数组，若nums[left] + nums[right] < target则left++，否则right--。直到两者相等则存在解。
3. 因为双指针的是针对有序组求目标值， 所以我们需要对输入数组排序，排序后的元素下标会发生改变，所以我们需要提前用哈希表记录元素下标，元素可能相同所以需要用集合存储。

理解了上面步骤代码遍不难写出

哈希表部分

```
Map<Integer, List<Integer>> map = new HashMap<>();
for (int i = 0; i < nums.length; i++) {
  if (map.containsKey(nums[i])) {
    map.get(nums[i]).add(i);
  } else {
    List<Integer> list = new ArrayList<>();
    list.add(i);
    map.put(nums[i], list);
  }
}
```

双指针部分，划个重点双指针代码**墙裂建议背诵默写**，数组问题必备结题法

```
        Arrays.sort(nums);
        int left = 0, right = nums.length - 1;
        while (left < right) {
            if (nums[left] + nums[right] == target) {
                return new int[]{map.get(nums[left]).get(0), nums[left] == nums[right] ? map.get(nums[right]).get(1) : map.get(nums[right]).get(0)};
            } else if (nums[left] + nums[right] > target) {
                right--;
            } else {
                left++;
            }
        }
```

复杂度分析：

时间复杂度O(nlogn)：双指针是基于有序数组从两端查找结果时间复杂度为O(n)，**在这里需要注意**数组的排序是通过Arrays.sort(nums)内部会根据数据大小选择插入排序或者双轴快速排序实现，具体可参考源码，也就是说在某些条件下,可以按线性时间对数据进行排序时间复杂度为O(nlogn)，因此最最终的时间复杂度为O(nlogn)

空间复杂度O(m)：使用了额外的map辅助空间，其中m为nums数组中不同元素的数量。

------

#### 解答三：一次遍历哈希表

该方法为本题的最优解，**需多记多背。**思路很简单很实用，类似于签到找人。如在大型网恋线下见面会上，到场的男男女女第一步是先问一下前台签到人员：我的网恋对象到了吗？前台签到人员查了下签到表，发现他的网恋对象到了则由工作人员带他到对方处，找人成功，否则进行签到安排就做。当他的对象到了就能直接通过签到表由工作人员直接带到他面前。

签到表即为哈希表用于记录，遍历元素，int num = target - nums[i];若num在map表中则找人成功，否则加入哈希表map.put(nums[i], i);

```
    public int[] twoSum(int[] nums, int target) {
        if (nums == null || nums.length < 2) {
            return new int[0];
        }

        Map<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int num = target - nums[i];
            if (map.containsKey(num)) {
                return new int[]{map.get(num), i};
            }
            map.put(nums[i], i);
        }
        return new int[0];
    }
```

复杂度分析

时间复杂度O(n)：针对n长度的数组只进行了一次遍历，且在表中进行的每次查找只花费O(1) 的时间。

空间复杂度O(n)：所需的额外空间取决于哈希表中存储的元素数量，该表最多需要存储 n 个元素。

### 同类题相关

若你反复AC，且充分理解了本题的3种解法，我敢保证下面三题解出来只是时间长短问题。

在这里还是提醒刷题的朋友，多看几次本文[跟我学：LeetCode刷题大法](https://www.toutiao.com/i6809940499399442956/?group_id=6809940499399442956)

LeetCode 15. 三数之和

LeetCode 16. 最接近的三树之和

LeetCode 18. 四数之和

### 技巧与心得

开一个github仓库用来记录各种题解，例如本人的https://github.com/ZhengYangxin/LeetCode

刷题不建议直接在网页上刷，在idea或者vscode中都提供了leecode的插件，谁用谁知道

遇到不会的题不要慌，直接看题解

文字类的表述不如视图类的直观，尽量找图文并茂精选图解，或者看leetcode国际站的大牛题解

图解依旧看着吃力，可以在哔哩哔哩或者YouTube上，通过leecode + 题号，搜索高收视率的视频观看

反复研读观看，相信我你最终会懂的

### 最后的最后

看到最后，我相信你对本题应该有不一样的感受了吧！come on 动起来，就是干！在此若有不对之处或建议等，留言告诉我；若有疑惑或者不一样的看法，也请告诉我，我们可以一起探讨学习！

我是Yangcy，该吃吃该喝喝，该学还得学，我们一起加油！

<img src="/images/follow_end_blog.jpg" style="zoom:30%;" />