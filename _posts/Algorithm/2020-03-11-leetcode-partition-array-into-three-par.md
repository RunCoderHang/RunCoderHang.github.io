---
title:  LeetCode：将数组分成和相等的三个部分
tags:
- LeetCode
- 算法
date:   2020-03-11 20:00:00
categories: 数据结构与算法
---

给你一个整数数组 A，只有可以将其划分为三个和相等的非空部分时才返回 `true` ，否则返回 `false` s。

形式上，如果可以找出索引 `i+1 < j` 且满足 `(A[0] + A[1] + ... + A[i] == A[i+1] + A[i+2] + ... + A[j-1] == A[j] + A[j-1] + ... + A[A.length - 1])` 就可以将数组三等分。

示例 1：

```
输出：[0,2,1,-6,6,-7,9,1,2,0,1]
输出：true
解释：0 + 2 + 1 = -6 + 6 - 7 + 9 + 1 = 2 + 0 + 1
```

示例 2：

```
输入：[0,2,1,-6,6,7,9,-1,2,0,1]
输出：false
```

示例 3：

```
输入：[3,3,6,5,-2,2,5,1,-9,4]
输出：true
解释：3 + 3 = 6 = 5 - 2 + 2 + 5 + 1 - 9 + 4
```

提示：

1. `3 <= A.length <= 50000`
2. `-10^4 <= A[i] <= 10^4`


## 思路

将数组分成和相等的三个部分，说明数组中元素的总和除以 3 就等于其中一部分元素之和。

使用一个巧妙的办法，定义一个 目标值 = 总和 / 3 ，和一个计数变量。每当遍历数组并且使数组元素相加时，所得到的和如果等于目标值，则计数 `+1` 。当计数量等于 3 时，返回 `true` 。

不过也有其他的案例，比如： `[1, -1, 1, -1, 1, -1, 1, -1]` 。 它的目标值为 0 ，计数会大于等于 3。因此，我们需要直接修改计数条件即可。

## 代码实现

```c
bool canThreePartsEqualSum(int* A, int ASize){
    int sum = 0;
    for(int i = 0; i < ASize; i++) {
        sum += A[i];
    }
    if(sum % 3 != 0) return false;
    int target = sum / 3;
    sum = 0;
    int count = 0;
    for(int i = 0; i < ASize; i++) {
        sum += A[i];
        if(target == sum) {
            count++;
            sum = 0;
        }
        if(count >= 3) return true;
    }
    return false;
}
```


```c++
class Solution {
public:
    bool canThreePartsEqualSum(vector<int>& A) {
        int sum = 0;
        for(auto& it : A)
            sum += it;
        if(sum % 3 != 0) return false;
        int target = sum / 3;
        int count = 0;     
        int currSum = 0;
        for(auto& it : A) {
            currSum += it;
            if(currSum == target ){
                currSum = 0;
                count++;
            }
        } 
        return count>=3;       
    }
};
```

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png">
</div>