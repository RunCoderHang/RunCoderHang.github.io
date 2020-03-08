---
title:  LeetCode每日一题（五）：分糖果
tags:
- LeetCode
- 算法
date:   2020-03-05 20:00:00
categories: 数据结构与算法
---

暴力解决，简单完事！

## 一、题目

排排坐，分糖果。

我们买了一些糖果 `candies`，打算把它们分给排好队的 `**n = num_people**` 个小朋友。

给第一个小朋友 1 颗糖果，第二个小朋友 2 颗，依此类推，直到给最后一个小朋友 `n` 颗糖果。

然后，我们再回到队伍的起点，给第一个小朋友 `n + 1` 颗糖果，第二个小朋友 `n + 2` 颗，依此类推，直到给最后一个小朋友 `2 * n` 颗糖果。

重复上述过程（每次都比上一次多给出一颗糖果，当到达队伍终点后再次从队伍起点开始），直到我们分完所有的糖果。注意，就算我们手中的剩下糖果数不够（不比前一次发出的糖果多），这些糖果也会全部发给当前的小朋友。

返回一个长度为 `num_people` 、元素之和为 `candies` 的数组，以表示糖果的最终分发情况（即 `ans[i]` 表示第 `i` 个小朋友分到的糖果数）。


## 二、思考

**暴力解决法：**

* 链表

这是我脑子一抽，突然想到了以前学过的循环链表。因为通过链表还是比较容易想到解决方法而且更容易理解（起码我是这么认为的）。

用循环链表中的每个节点代表每个小盆友。节点中定义所得糖果数目信息变量，记录每个人的得到的糖果数量。在一次次循环中进行相加计算。

而根据时间复杂度和空间复杂度的大小表明：这根本不是一个好的方法...

* 数组

对比他人答案时候发现原来数组也是可以循环的。使用的就是此语句：

`index = index % num_people;`

这样根据索引值来找小盆友，并将得到的糖果数量存放数组中。同样是暴力方法解决，数组方式的空间与时间耗费总比链表好多了。

## 三、代码实现

**厚着脸皮把链表贴出来了**

```c
typedef struct LNode {      // 每个节点代表一个小盆友
    int candyNum;           // 记录小盆友得到的糖果数
    struct LNode* next;
} LinkList;

LinkList* CreateList(int num_people) {  // 建立没有头结点的循环链表
    LinkList *s, *r;
    LinkList *L = (LinkList*)malloc(sizeof(LinkList));
    L -> candyNum = 0;
    r = L;
    for(int i = 2; i <= num_people; i++) {
        s = (LinkList*)malloc(sizeof(LinkList));
        s -> candyNum = 0;
        r -> next = s;
        r = s;
    }
    r -> next = L; 
    return L;
}

int* distributeCandies(int candies, int num_people, int* returnSize){
    // 建立循环链表
    LinkList* L = CreateList(num_people);
    LinkList* p = L;

    int getCandies = 0;     // 记录小盆友应得的糖果
    int* ans = (int*)malloc(num_people*sizeof(int));
    if(candies<=0){
        *returnSize = num_people;
        return ans;
    }
    while(candies > getCandies) { // 比较剩余糖果数和应得的糖果数
        getCandies++;
        candies -= getCandies;
        p -> candyNum += getCandies;
        p = p -> next;
    }   
    p -> candyNum += candies;  // 当剩余的糖果不够发了，那就给循环到的小盆友吧

    for(int i = 0; i < num_people; i++) { 
        ans[i] = L -> candyNum;
        L = L -> next;
    }
    *returnSize = num_people; // 不清楚这个用来干什么的，但是不写会报错。
    return ans;
}
```

**数组**

```c
int* distributeCandies(int candies, int num_people, int* returnSize){
    int* ans=(int*)malloc(sizeof(int)*num_people);
    for(int i = 0; i < num_people; i++){
        ans[i] = 0;
    }
    if(candies <= 0){
        *returnSize = num_people;
        return ans;
    }
    int getCandies = 0;
    int index = 0;      // 小盆友的索引值
    while(candies > getCandies){
        getCandies++;
        candies -= getCandies;
        ans[index] += getCandies;
        index = index % num_people;
        index++;
    }
    ans[index%num_people] += candies;

    *returnSize = num_people;
    return ans;
}
```

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png"></img>
</div>