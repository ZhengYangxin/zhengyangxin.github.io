---
title: 跟我学：LeetCode刷题之2.两数相加
copyright: true
date: 2020-04-03 06:23:47
tags: [Leetcode，两数相加]
author: Yangcy
summary: 两数相加、多解法分析
category: 算法
---

### 导读

本题是LeetCode第二题，难度标记为中等。此题为链表题，不出意外遇到链表题都是画图解题，因为链表题的核心是在符合条件下的链表next指针的变化，除了链表的增删改查之外，如何**反转链表的代码，**我**墙裂建议你先**理解背诵，刷链表题基础必备代码，如下

```
pubic ListNode reverse(ListNode head) {
    ListNode pre = null;
    ListNode cur = head;

    while(cur != null){
        ListNode next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
}
```

这里通过一张思维导图帮助大家整理思路。思维导图一方面罗列了本文的大纲，另一方面也是希望读者朋友可以保存图片至自己的复习资料里，以便随时通过该大纲去复习。

![跟我学：LeetCode刷题之2.两数相加](http://p1.pstatp.com/large/pgc-image/f6834647753d4094b9ac461b3ba45cc6)

### 看题总结

给出**两个 非空 的链表**用来表示**两个非负的整数**。其中，它们各自的位数是按照 **逆序 的方式存储的**，并且它们的每个节点只能存储 **一位** 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，**这两个数都不会以 0 开头**。

**示例：**

```
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```

Related Topics

链表

数学

通过对题目的阅读，我们可以得出以下几个结论

- 题目类型为链表
- 用两个链表表示两个整数，整数在链表中的存储方式是逆序的，通过链表实现两整数的相加操作，返回值也是用逆序的链表存储的整数。
- 非空链表，非负数整数，除了0之外整数不会以0开头，输入是合法的

### 多解法及复杂度分析

#### 解法一：非递归实现

1. 定义一个哑结点dummy（可以避免对头结点的处理），指向当前节点cur，然后定义一个保存进位数值的 carry（因为两数相加可能会产生进位，便用 carry 来保存）
2. 当 l1 或 l2 不为空时进行遍历，即二者都为空时才退出循环，
3. 通过设置默认值为 0 的 x和 y，使得当二者中有一个为空时，使用 0 来与另一位相加
4. 得到 sum 后，将 sum 的个位数连接到链表中，并将十位数赋给 carry,
5. 移动 cur 使其一直指向链表的尾部

最后要判断 carry 是否大于 0，因为例如 (1 -> 2 -> 3) + (7 -> 8 -> 9) 会得到结果 8 -> 0 -> 3 -> 1，所以在最后判断 l1 和 l2 都已经为空了，但是此时 carry 大于 0，所以要将这最后一位数加入到链表尾部

```
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;
        int carry = 0; // 保存进位数值的 carry
        while (l1 != null || l2 != null) {
          	// 用0补齐，链表长度
            int x = l1 != null ? l1.val : 0;
            int y = l2 != null ? l2.val : 0;
            int sum = x + y + carry;
            carry = sum / 10;
            ListNode newNode = new ListNode(sum % 10);

            curr.next = newNode;
            l1 = l1 != null ? l1.next : null;
            l2 = l2 != null ? l2.next : null;
        }
  
  			// 存在两链表长度相等，且最高位计算和大于10，需进位创建新节点
        if (carry > 0) {
            curr.next = new ListNode(carry);
        }

        return dummy.next;
    }
```



------

#### 解法二：递归实现

首先我们需要理解递归，**墙裂建议背诵默写**递归的模板代码，递归解法的套路如下

```
public void recur(int level, Param param) {
  	// 递归终止条件
    if (level > MAX_VALUE) {
        //process result
        return;
    }

    // 当前层处理处理
    process(param);

    // 进入下一层递归
    recur(level, newParam);

    // 非必要的数据恢复
}
```

这题使用递归很好理解，两链表的节点从左往右进行相加，进位，直到两链表都到达尾部。

1. 递归终止条件：两链表都到达尾部
2. 当前层处理：链表节点相加计算和，得到余数，及carry进位
3. 进入下一次递归：传入新节点及进位值。

最后代码如下

```
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        return add(l1, l2, 0);
    }

    private ListNode add(ListNode a, ListNode b, int carry) {
        // terminator 终止条件
        if (a == null && b == null) {
            return carry > 0 ? new ListNode(1) : null;
        }

        // current logic  当前层逻辑处理
        // 如果当前节点a,b其中一个为null，则赋值为ListNode(0)，方便计算
        a = a == null ? new ListNode(0) : a;
        b = b == null ? new ListNode(0) : b;

        // 当前两链表节点的和
        int sum = a.val + b.val + carry;
        carry = sum / 10;
        // 将取模结果保存在第一个链表中
        a.val = sum % 10;
        // driil down   进入下一层递归
        a.next = add(a.next, b.next, carry);
        return a;
    }
```

### 同类型题相关

若你反复AC，且充分理解了本题的两种题解，我敢保证下面这题解出来只是时间长短问题。

在这里还是提醒刷题的朋友，多看几次本文[跟我学：LeetCode刷题大法](https://www.toutiao.com/i6809940499399442956/?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956)

LeetCode 445. 两数相加 II

### 技巧与心得

1. 开一个github仓库用来记录各种题解
2. 在idea或者vscode中都提供了leecode的插件刷题，方便调试断走流程
3. 遇到不会的题不要慌，稳住心态直接看题解
4. 文字类的表述不如视图类的直观，图文并茂精选图解优先或者参考leetcode国际站的大牛题解
5. 图解依旧看着吃力也无妨，可以在哔哩哔哩或者YouTube上，通过leecode + 题号，搜索高收视率的视频观看
6. 多看几次本文[跟我学：LeetCode刷题大法](https://www.toutiao.com/i6809940499399442956/?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956?group_id=6809940499399442956)，相信我你肯定能越刷越自信

### 最后的最后

看到最后，我相信你对本题应该有不一样的感受了吧！在此若有不对之处或建议等，留言告诉我；若有疑惑或者不一样的看法，也请告诉我，我们可以一起探讨学习！

我是Yangcy，该吃吃该喝喝，该学还得学，我们一起加油！

<img src="/images/follow_end_blog.jpg" style="zoom:30%;" />