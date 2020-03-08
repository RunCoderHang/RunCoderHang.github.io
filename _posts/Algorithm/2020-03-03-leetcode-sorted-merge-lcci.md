---
title:  LeetCode每日一题（三）：合并排序的数组
tags:
- LeetCode
- 算法
date:   2020-03-03 20:00:00
categories: 数据结构与算法
---

给定两个排序后的数组 `A` 和 `B` 合并。实现的方法有很多，也是非常常见的操作数组题目。

## 一、题目

给定两个排序后的数组 `A` 和 `B` ，其中 `A` 的末端有足够的缓冲空间容纳 `B` 。 编写一个方法，将 `B` 合并入 `A` 并排序。

初始化 `A` 和 `B` 的元素数量分别为 `m` 和 `n` 。

示例:

```
输入:
A = [1,2,3,0,0,0], m = 3
B = [2,5,6],       n = 3

输出: [1,2,2,3,5,6]
```

## 二、思考

1. 直接合并后排序。
    * `c++` 实现：使用 `c++` 的 **sort** 实现排序，而 **sort** 排序方法是类似于快排的方法，时间复杂度为 **n*log2(n)**
    * `c` 实现：使用冒泡排序或者快速排序。
2. 双指针
    * 对两个数组的元素从第一个开始比较，比较结果添加至新的数组中。
    * 由于需要新的数组，所以要开辟新的数组空间。
3. 逆双指针
    * 不需要开辟新的空间。
    * 对两个数组从后往前比较，将最大的数依次放置最长数组的末尾。

## 三、代码实现

**直接合并后排序**

```c
/*
* 快速排序
*/
int Partition(int A[], int low, int high) {
    int pivot = A[low];                         // 当前表中第一个元素设为枢轴值，对表进行划分
    while(low < high) {
        while(low < high && A[high] >= pivot) --high;
        A[low] = A[high];                       // 将比枢轴值小的元素移动到左端
        while(low < high && A[low] <= pivot) ++low;
        A[high] = A[low];                       // 将比枢轴值大的元素移动到右端
    }
    A[low] = pivot;                             // 枢轴元素存放到最终位置
    return low;                                 // 返回存放枢轴值的最终位置
}
void QuickSort(int A[], int low, int high) {
    if(low < high) {                            // 递归跳出的条件                  
                                                // Partition 是划分操作，将表划分为两个子表
        int pivotpos = Partition(A, low, high); // 划分
        QuickSort(A, low, pivotpos - 1);        // 依次对两个子表进行递归排序
        QuickSort(A, pivotpos + 1, high);
    }
}
void merge(int* A, int ASize, int m, int* B, int BSize, int n){
    int i = 0;
    for(int i = 0; i != n; i++)                 // 将两个数组合并
        A[m + i] = B[i];
    QuickSort(A, 0, m + n - 1);                 // 快速排序
}
```

```c++
/*
* 使用C++中的sort排序
*/
class Solution {
public:
    void merge(vector<int>& A, int m, vector<int>& B, int n) {
        for(int i = 0; i != n; i++)
            A[m + i] = B[i];
        sort(A.begin(), A.end());
    }
}
```

* 时间复杂度：O((m+n)log(m+n)))
* 空间复杂度：O(log(m+n)))

**双指针**

```c
void merge(int* A, int ASize, int m, int* B, int BSize, int n) {
    int pa = 0, pb = 0;                 // 分别指向两个数组起始数据
    int sorted[m + n];                  // 开辟新的数组存放有序序列
    int cur;                            // 放置新的数组的中间变量
    while(pa < m || pb < n) {
        if(pa == m)                     // 遍历完数组A的有效数据
            cur = B[pb++];
        else if(pb == n)                // 遍历完数组B的有效数据
            cur = A[pa++];
        else if(A[pa] < B[pb])          // 比较出最小的元素
            cur = A[pa++];
        else
            cur = B[pb++];
        sorted[pa + pb - 1] = cur;      // 将元素有序的放置新数组中
    }
    for(int i = 0; i != m + n; i++) {
        A[i] = sorted[i];
    }
}
```

* 时间复杂度：O(m+n)
* 空间复杂度：O(m+n)

**逆双指针**

```c
void merge(int* A, int ASize, int m, int* B, int BSize, int n) {
    int pa = m - 1, pb = n - 1;         // 分别指向两个数组末端的有效数据e(e != 0)
    int tail = m + n - 1;               // 指向最长数组的末端
    int cur;                            // 替换末端数据的中间变量
    while(pa >= 0 || pb >= 0) {
        if(pa == -1)                    // 当A的有效数据遍历完毕
            cur = B[pb--];
        else if(pb == -1)               // 当B的有效数据遍历完毕
            cur = A[pa--];
        else if(A[pa] > B[pb])          // 比较出最大的元素
            cur = A[pa--];
        else
            cur = B[pb--];
        A[tail--] = cur;                // 将最大元素放置A的末端
    }
}
```

* 时间复杂度：O(m+n)
* 空间复杂度：O(1)

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png">
</div>