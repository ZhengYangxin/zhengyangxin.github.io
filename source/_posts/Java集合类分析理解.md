---
title: Java集合类分析理解
copyright: true
date: 2018-01-25 17:32:12
tags: ["集合"]
category: "Java"
---
[Java集合面试总结](http://blog.csdn.net/csdn_terence/article/details/78379878 "Optional title")
## Collecton 和 Map
### Collection
包括 Set List Queue
<!-- more -->
#### 主要分析
1. ArrayList
 * 动态数组
 * capacity 扩容机制  1.5倍， 初始 java1.8 : 10 ;android 21 :12
 * 内部元素变动 System.copyarray();
 * 线程不安全

2. LinkList
双向链表
Node()函数，该函数以O(1/2)的性能去获取一个节点
链表操作
线程不安全

3. [HashMap](http://www.importnew.com/20386.html "Optional title")
 * hash()方法 (h = key.hashCode()) ^ (h >>> 16) able的长度都是2的幂，因此index仅与hash值的低n位有关 [计算方式](http://blog.csdn.net/fan2012huan/article/details/51097331"Optional title")
 * tableSizeFor() 找到大于等于initialCapacity的最小的2的幂
 * Node[] tab哈希桶数组.哈希冲突，开放地址和链地址法
 * 根据key获取哈希桶数组索引位置
 * tab 长度为2的幂次
 * threshold 所能容纳的key-value对极限
 * Map m = Collections.synchronizeMap(hashMap)实现同步
 * 初始 capacity 16 loadFactor 0.75

4. [TreeMap](http://www.iqiyi.com/v_19rro6v558.html?vfm=m_312_shsp "Optional title")
 * 红黑树的定义，节点非红即黑，根节点为黑色，不能连续的红色，任意节点到末端的路径黑色个数相同
 * 红黑树的平衡调整，颜色和结构
 * 继承了SortMap ，put()函数会做比较
 * 寻找后继节点，中序遍历

5. LinkHashMap
* 与HashMap类似，不过保证了put顺序
* 主要实现了afterNodeAccess(),afterNodeInsert(),afterNoderemoval三个方法
