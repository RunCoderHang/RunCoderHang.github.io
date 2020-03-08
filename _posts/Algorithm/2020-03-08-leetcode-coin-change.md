---
title:  LeetCode每日一题（八）：零钱兑换
tags:
- LeetCode
- 算法
date:   2020-03-08 20:00:00
categories: 数据结构与算法
---

**DP** 题...今天难倒我了，一下午都在思考。没错，是我太菜了。

## 一、题目

给定不同面额的硬币 `coins` 和一个总金额 `amount` 。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。

示例 1:

```
输入: coins = [1, 2, 5], amount = 11
输出: 3 

解释: 11 = 5 + 5 + 1
```

示例 2:

```
输入: coins = [2], amount = 3
输出: -1
```

说明:
你可以认为每种硬币的数量是无限的。

## 二、思路

**dp** 动态规划问题...触及到我的知识盲区了。我还只是一只小 rookie 。

原本使用暴力解法，然而我还不知道对不对就直接 time out 了。滚去研究官方和大佬们的答案了，当然还有这个 **dp** 。

## 三、代码实现

**自下而上**

```c
#define min(a,b) ((a) < (b) ? (a) : (b))
int coinChange(int* coins, int coinsSize, int amount){
    int i, j, dp[amount + 1];
    dp[0] = 0;
    for(i = 1;i < amount + 1;i++)
    {
        dp[i] = INT_MAX;
        for(j = 0;j < coinsSize;++j)
        {
            if(i >= coins[j])
                dp[i] = min(dp[i], dp[i-coins[j]] + 1);
        }
    }
    return dp[amount] == INT_MAX ? -1 : dp[amount];
}
```

**官方答案** ，拿小本本记下来。

```c++
/*
* 动态规划-自上而下
* 
* 作者：LeetCode-Solution
* 链接：https://leetcode-cn.com/problems/coin-change/solution/322-ling-qian-dui-huan-by-leetcode-solution/
* 来源：力扣（LeetCode）
*/
class Solution {
    vector<int>count;
    int dp(vector<int>& coins, int rem) {
        if (rem < 0) return -1;
        if (rem == 0) return 0;
        if (count[rem - 1] != 0) return count[rem - 1];
        int Min = INT_MAX;
        for (int coin:coins) {
            int res = dp(coins, rem - coin);
            if (res >= 0 && res < Min) {
                Min = res + 1;
            }
        }
        count[rem - 1] = Min == INT_MAX ? -1 : Min;
        return count[rem - 1];
    }
public:
    int coinChange(vector<int>& coins, int amount) {
        if (amount < 1) return 0;
        count.resize(amount);
        return dp(coins, amount);
    }
};
```

```c++
/*
* 动态规划-自下而上
* 
* 作者：LeetCode-Solution
* 链接：https://leetcode-cn.com/problems/coin-change/solution/322-ling-qian-dui-huan-by-leetcode-solution/
* 来源：力扣（LeetCode）
*/
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        int Max = amount + 1;
        vector<int> dp(amount + 1, Max);
        dp[0] = 0;
        for (int i = 1; i <= amount; ++i) {
            for (int j = 0; j < (int)coins.size(); ++j) {
                if (coins[j] <= i) {
                    dp[i] = min(dp[i], dp[i - coins[j]] + 1);
                }
            }
        }
        return dp[amount] > amount ? -1 : dp[amount];
    }
};


```

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png">
</div>