---
title:  LeetCode：买卖股票的最佳时机
tags:
- LeetCode
- 算法
date:   2020-03-09 20:00:00
categories: 数据结构与算法
---

今天没有什么难度，求最大差值。但是解题的方法有很多，不仅仅文中的两种。

## 一、题目

给定一个数组，它的第 `i` 个元素是一支给定股票第 `i` 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票），设计一个算法来计算你所能获取的最大利润。

注意你不能在买入股票前卖出股票

**示例 1:**

```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
```

**示例 2:**

```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```


## 二、思路

#### 暴力解决法

两遍 `for` 循环，找出两个数字之间的最大差值，即最大利润。

内层 `for` 循环中的数组下标总是在外层数组下标之后。也就是拿后面的卖出价格减去前面的买入价格，求出最大利润。

时间复杂度： **O(n^2)**
空间复杂度： **O(1)**

#### 一次遍历

遍历一遍数组。

找到第 `i` 天买入股票的最低价格，在找最低价格的同时求出最大利润。有人可能举出 `[6, 2, 5, 1, 3]` 的例子：你找到了最低价格为 1 ，但是你的利润为 `3-1=2` 不是最大的啊。

首先，这里的 `i` , 是指的是从数组的第一个元素开始找，当找到第 `i` 天的股票价格在 `[i, i+n]` 的区间中最低，则求出最大利润 `maxprofit` 。

如果，在第 `i + a` 天的股票价格比第 `i` 天的价格小，从而改变区间。在第 `[i+a, i+a+n]` 的区间中求出 `maxprofit` 。并且与之前的利润作对比，返回最大利润。

时间复杂度： **O(n)**
空间复杂度： **O(1)**

## 三、代码实现

**暴力解决**

```c
// 暴力求解
int maxProfit(int* prices, int pricesSize){
    int profit = 0, maxprofit = 0;
    for(int i = 0; i < pricesSize - 1; i++) {
        for(int j = i+1; j < pricesSize; j++) {
            profit = prices[j] - prices[i];
            if(maxprofit < profit) {
                maxprofit = profit;
            }
        }
    }
    return maxprofit;  
}
```

**一次遍历**

**C**

```c
// 一次遍历
int maxProfit(int* prices, int pricesSize){
    int minprice = INT_MAX;
    int maxprofit = 0;
    for(int i = 0; i < pricesSize; i++) {
        if(minprice > prices[i]) {
            minprice = prices[i];
        } else if(prices[i] - minprice > maxprofit) {
            maxProfit =  prices[i] - minprice;
        }
    }
    return maxprofit;
}
```

**C++**

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int minprice = INT_MAX;
        int maxprofit = 0;
        for (int price : prices) {
            minprice = min(price, minprice);
            maxprofit = max(maxprofit, price - minprice);
        }
        return maxprofit;
    }
};
```

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png">
</div>