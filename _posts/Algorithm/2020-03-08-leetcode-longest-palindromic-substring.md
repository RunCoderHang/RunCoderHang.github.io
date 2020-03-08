---
title:  LeetCode：最长回文子串
tags:
- LeetCode
- 算法
date:   2020-03-08 20:00:00
categories: 数据结构与算法
---

**OJ** 上的题就是参考这里的方法。回文字符串也是经常出现的题目。

## 一、题目

给定一个字符串 `s` ，找到 `s` 中最长的回文子串。你可以假设 `s` 的最大长度为 `1000` 。

示例 1：

```
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
```

示例 2：

```
输入: "cbbd"
输出: "bb"
```

## 二、思路

**中心扩展法**

回文中心两侧互为镜像。因此可以从它的中心展开，并且只有 2n-1 个这样的中心。

为什么是 2n-1 个？因为回文中的中心字符串长度可能有两种：奇数与偶数。例如：

1. ada ：只有 d 这一个
2. adda：有 dd 这两个

有人指出：addda 这三个 ddd 呢？实质上，这其实就是中间只有一个 d 的奇数个。之后，就在中间这个 d 或者 dd 为中心，向两边扫描找出相同的字符。由此找到回文字符串。

* `start = i- (mlen - 1) / 2; // 回文字符串开始字符的索引值`
* `end = i + mlen / 2; // 回文字符串结束字符的索引值`
* `len = max(max(len1, len2), len); // 最长回文字符串的长度`

## 三、代码实现

```c++
class Solution {
public:
    string longestPalindrome(string s) {
        int start = 0;
        int end = 0;
        int len = 0;
        for(int i = 0; i < s.size(); i++) {
            int len1 = expendAroundCenter(s, i, i);
            int len2 = expendAroundCenter(s, i, i + 1);
            len = max(max(len1, len2), len);
            if(len>end-start+1)
            {
                start=i-(len-1)/2;
                end=i+len/2;
            }
        }
        return s.substr(start, len);
    }

    int expendAroundCenter(string s, int left, int right) {
        int L = left, R = right;
        while(L>=0 && R<s.size() && s[L] == s[R]) {
            L--;
            R++;
        }
        return R - L - 1;
    }
};
```

**复杂度：**

* 时间复杂度： O(n^2)
* 空间复杂度： O(1)

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png">
</div>