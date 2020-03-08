---
title:  LeetCode每日一题（六）：和为s的连续正数序列
tags:
- LeetCode
- 算法
date:   2020-03-06 20:00:00
categories: 数据结构与算法
---

## 一、题目

输入一个正整数 `target` ，输出所有和为 `target` 的连续正整数序列（至少含有两个数）。

序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

**示例 1：**

```
输入：target = 9
输出：[[2,3,4],[4,5]]
```

**示例 2：**

```
输入：target = 15
输出：[[1,2,3,4,5],[4,5,6],[7,8]]
```

限制：

* `1 <= target <= 10^5`

## 二、思考

1. 枚举 + 暴力

当然，如果你想直接从 1 遍历至 target ，写两层循环，也是可以暴力解决此问题。不过时间复杂度就相对比较大了。

根据官方解法：枚举每个正整数为起点，判断以它为起点的序列之和 sum 是否等于 target 即可。当然，我们不可能从 1 至 target 挨个枚举出来。所以给出枚举的上限： **`target / 2`** 。在 C 中遇到奇数则是向下取整。

2. 双指针

其实就是两个指针指向闭区间的区间端点，而这个闭区间内数之和就是 target。

设闭区间为 `[l, r]`，我们可以列出公式（等差数列求和公式）

**`target = (l + r) * (r - l + 1) / 2`**

使用双指针时，总会遇到所求和 **sum** 与 **target** 大小不等的时候。

当 **sum < target** 时，**r++** ，是为了扩大区间长度。

当 **sum > target** 时，**l++** ，是为了区间向左移动，重新求和。 


## 三、代码实现

**枚举 + 暴力**

```c
 // 枚举 + 暴力 
 int** findContinuousSequence(int target, int* returnSize, int** returnColumnSizes){ 
     int sum = 0; 
     int limit = target / 2; // 向下取整 
     int** res = (int**)malloc(sizeof(int*)*target); // 返回结果的二维数组 
     int* cols = (int*)malloc(sizeof(int)*target); 
     int dx = 0; // 二维数组中行数 
     for(int i = 1; i < limit; i++)  
     { 
         for(int j = i; ; j++)  
         { 
             sum += j; 
             if (sum > target) 
             { 
                 sum = 0; 
                 break; 
             } else if (sum == target) 
             { 
                 cols[dx] = j - i + 1;   // 返回最终二维数组的列数 
                 res[dx] = (int*)malloc(sizeof(int)*(j-i+1)); // 这里是在创建二维数组空间 
                 for (int num = i, dy = 0; num <= j; num++, dy++) 
                 { 
                     res[dx][dy] = num;  
                 } 
                 dx++; 
                 sum++; 
                 break; 
             } 
         } 
     } 
     *returnSize = dx; 
     *returnColumnSizes = cols; 
     return res; 
 } 
```

**双指针**

```c
 // 双指针 
 int** findContinuousSequence(int target, int* returnSize, int** returnColumnSizes){ 
     int sum = 0; 
     int** res = (int**)malloc(sizeof(int*)*target); 
     int* cols = (int*)malloc(sizeof(int)*target); 
     int r = 2, l = 1; 
     int dx = 0; 
     while(l < r) { 
         sum = (l + r) * (r - l + 1) / 2;  // 根据公式求和 
         if (sum == target) { 
             cols[dx] = r - l + 1; 
             res[dx] = (int*)malloc(sizeof(int)*(r-l+1)); 
             for(int dy = 0, num = l; num <= r; dy++, num++) { 
                 res[dx][dy] = num; 
             } 
             dx++; 
             l++; 
         } 
         else if(sum < target) { 
             r++;    // 如果结果小于target，则区间长度扩大 
         } else { 
             l++;    // 如果结果大于target，则区间向左移动 
         } 
     } 
     *returnSize = dx; 
     *returnColumnSizes = cols; 
     return res; 
 } 
```

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png">
</div>