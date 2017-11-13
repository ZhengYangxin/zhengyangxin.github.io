---
layout: pot
title: Pascal's Triangle
date: 2017-11-13 10:22:56
tags: [leetCode,算法]
category: "算法"
---
题目链接:[Pascal's Triangle](https://leetcode.com/problems/pascals-triangle/description/ "Optional title")

Given numRows, generate the first numRows of Pascal's triangle.
For example, given numRows = 5,
Return
```
[
     [1],
    [1,1],
   [1,2,1],
  [1,3,3,1],
 [1,4,6,4,1]
]
```

这道题比较简单，属于基础的数组操作。知道规律就好做了，它的规律如下：
1.每行的元素个数等于行数
2.每行的第一个元素和最后一个元素等于1
3.第三行起，每行除第一个和最后一个元素外之间的元素第i行第k个元素 = 行数i-1行第k个元素 + 行数i-1行第k-1元素
4.隐藏条件：当行数i和元素j的下表相等时 ，i行的第j个元素等于1
具体代码如下：

```
//方法一
public static List<List<Integer>>  generate(int numRows){
    List<List<Integer>> triangle = new ArrayList<>();
    if(numRows <= 0){
        return triangle;
    }

    for (int i = 0; i < numRows; i++) {
        List<Integer> row = new ArrayList<>();
        for (int j = 0; j < i+1; j++) {
            if(j ==0 || j == i){
                row.add(1);
            }else {
                row.add(triangle.get(i-1).get(j-1) + triangle.get(i-1).get(j));
            }
        }
        triangle.add(row);
    }
    return triangle;
}


//方法二
public static int[][]  pascal(int rowNums){
    if(rowNums<=0)
        return new int[1][1];

    int[][] arr = new int[rowNums][];
    for (int i = 0; i < rowNums; i++){
        arr[i] = new int[i+1];
        for (int j = 0; j < i+1; j++){
            if(j == 0 || j == i){
                arr[i][j] = 1;
            }else {
                arr[i][j] = arr[i-1][j-1] + arr[i-1][j];
            }
        }
    }
    return arr;
}
``` 
算法时间复杂度应该是O(1+2+3+...+n)=n(n-1)/2=O(n^2)，空间上只需要二维数组来存储结果，不需要额外空间。