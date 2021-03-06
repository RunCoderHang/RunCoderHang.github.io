---
title:  ZJUTOJ-1001-密码截获
tags:
- ZJUTOJ
- 算法
date:   2020-02-29 20:00:00
categories: 数据结构与算法
---

关于获取回文字符串的题目。

## Description:

Catcher是MCA国的情报员，他工作时发现敌国会用一些对称的密码进行通信，比如 `ABBA`，`ABA`，`A`，`123321` 等，但是他们有时会在开始或结束时加入一些无关的字符以防别国破解。比如进行下列变化 `ABBA->12ABBA` , `ABA->ABAKK` , `123321->51233214`　。因为截获的串太长了，而且存在多种可能的情况（ `abaaab` 可看作是 `aba` 或 `baaab` 的加密形式），Cathcer的工作量实在是太大了，他只能向电脑高手求助，你能帮Catcher找出最长的有效密码串吗？

## Input:

测试数据有若干行字符串，包括字母（字母区分大小写），数字，符号。

## Output:

与输入相对应每一行输出一个整数，代表最长有效密码串的长度。

## Sample Input:

```
ABBA
12ABBA
A
ABAKK
51233214
abaaab
```

## Sample Output:

```
4
4
1
3
6
5
```

## Solve:

**中心扩展法**

回文中心两侧互为镜像。因此可以从它的中心展开，并且只有 2n-1 个这样的中心。

为什么是 2n-1 个？因为回文中的中心字符串长度可能有两种：奇数与偶数。例如：

1. ada ：只有 d 这一个
2. adda：有 dd 这两个

有人指出：addda 这三个 ddd 呢？实质上，这其实就是中间只有一个 d 的奇数个。之后，就在中间这个 d 或者 dd 为中心，向两边扫描找出相同的字符。由此找到回文字符串。

* `start = i- (mlen - 1) / 2; // 回文字符串开始字符的索引值`
* `end = i + mlen / 2; // 回文字符串结束字符的索引值`
* `len = max(max(len1, len2), len); // 最长回文字符串的长度`

```c++
#include<iostream>
#include<string>
using namespace std;
int expendAroundCenter(string s, int left, int right);
int main() {
    string s;
    while(cin >> s) {
        int start = 0; // 回文开始的字符索引值
        int end = 0; // 回文结束的字符索引值
        int len = 0; // 回文的长度
        for(int i = 0; i < s.size(); i++) {
            int len1 = expendAroundCenter(s, i, i); // 中心字符为奇数时
            int len2 = expendAroundCenter(s, i, i + 1); // 中心字符为偶数时
            len = max(max(len1, len2), len);
            if(len>end-start+1) {
                start=i-(len-1)/2;
                end=i+len/2;
            }
        }
        cout << s.substr(start, len) << endl;
        cout << s.substr(start, len).size() << endl;
    }
    return 0;
}
// 中心扩展法的函数
int expendAroundCenter(string s, int left, int right) {
    int L = left, R = right;
    while(L>=0 && R<s.size() && s[L] == s[R]) {
        L--;
        R++;
    }
    return R - L - 1;
}
```

**复杂度：**

* 时间复杂度： O(n^2)
* 空间复杂度： O(1)

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png">
</div>