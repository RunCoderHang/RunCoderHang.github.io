---
title:  LeetCode：两数之和
tags:
- LeetCode
- 算法
date:   2019-02-29 20:00:00
categories: 数据结构与算法
---

给定一个整数数组 `nums` 和一个目标值 `target`，请你在该数组中找出和为目标值的那 **两个** 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

**示例：**

```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

**解决：**

#### 方法一：暴力破解法

1. 外循环遍历数组的每一个数 x
2. 内循环中则查找与 target-x 相等的元素
3. 最后返回内外循环中对应元素的索引值
4. 时间复杂度： O(n^2)
5. 空间复杂度： O(1)

```java
public class Demo01TwoSum {
    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            for (int j = 0; j < nums.length; j++) {
                if (nums[j] == target - nums[i]) {
                    return new int[] {i, j};
                }
            }
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}

```

#### 方法二：两遍哈希表

1. 哈希表是将空间换取时间，进行快速查找
2. 首先，将数组中每个元素和它的索引值添加到哈希表中
3. 然后，在表中查找与 target-num[i] 相等的元素
4. 最后，返回查找到元素的索引值
5. 时间复杂度： O(n)
6. 空间复杂度： O(n)

```java
/**
 * 数组查询的时间复杂度：O(n)
 * 链表查询的时间复杂度：O(n)
 * 哈希表查询的时间复杂度：O(1)
 */
public class Demo02TwoSum {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            map.put(nums[i], i);
        }
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.containsKey(complement) && map.get(complement) != i) {
                return new int[] {i, map.get(complement)};
            }
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}
```

#### 方法三：一遍哈希表

* 根据两遍哈希表，进行迭代并将元素插入到表中的同时，还会回过头来检查表中是否已经存在当前元素所对应的目标元素。如果存在，则将其返回
* 时间复杂度： O(n)
* 空间复杂度： O(n)

```java
public class Demo03TwoSum {
    public int[] twoSum(int num[], int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < num.length; i++) {
            int complement = target - num[i];
            if (map.containsKey(complement)) {
                return new int[] {i, map.get(complement)};
            }
            map.put(num[i], i);
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}
```

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png"></img>
</div>