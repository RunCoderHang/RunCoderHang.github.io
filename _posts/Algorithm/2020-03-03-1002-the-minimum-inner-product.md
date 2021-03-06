---
title:  ZJUTOJ-1002-最小内积
tags:
- ZJUTOJ
- 算法
date:   2020-03-03 20:00:00
categories: 数据结构与算法
---

向量是几何中的一个重要概念。

考虑两个向量 `v1=(x1,x2,...,xn)` 和 `v2=(y1,y2,...,yn)`，向量的内积定义为 `x1*y1 + x2*y2 + ... + xn*yn`

例如向量 (1,9,8,8) 和 (0,9,1,1) 的内积是 1×0+9×9+1×8+1×8=97。

下面我们考虑这样一个问题，如果我们能够任意的重新排列 `v1` 和 `v2` 中的分量（但是不能修改，删除和添加分量），然后再计算内积。显然这样计算的内积取决于选择的重排方式。

我们现在要问的是，通过重排向量中的分量，所能够获得的最小的内积是多少呢？

## Input:

输入数据包含3行。

第一行是一个整数 N，N<=100，代表了向量的维数。

第二行是N个非负整数，给出了 `v1` 中的元素，每个整数都在32位整数的范围内，用一个空格隔开。

第二行是N个非负整数，给出了 `v2` 中的元素，每个整数都在32位整数的范围内，用一个空格隔开。

输入一系列由空格隔开的 a、b 组成的整数对，每行一对整数，0 0 表示结束输入，并且不需要输出。

## Output:

输出一个整数，代表了通过重排向量中的分量，所能够获得的最小内积值。数据保证了最后结果在32位整数的范围内。

## Sample Input:

```
5
1 2 3 4 5
1 0 1 0 1
```

## Sample Output:

```
6
```

## Solve:

```c++
#include <iostream>
#include <vector>
#include <math.h>
#include <algorithm>
using namespace std;

bool compare(int a, int b) {
    return a > b;   //降序排列，如果改为return a<b，则为升序
}

int main() {
    int n = 0;
    while(cin >> n) {
        long sum = 0;
        int *a = new int[n];
        int *b = new int[n];
        for (int i = 0; i < n; i++) {
            cin >> a[i];
        }
        for(int i = 0; i < n; i++) {
            cin >> b[i];
        }
        sort(a, a + n);
        sort(b, b + n, compare);
        for(int i = 0; i < n; i++) {
            sum += a[i] * b[i];
        }
        cout << sum << endl;
    }
}
```

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png">
</div>