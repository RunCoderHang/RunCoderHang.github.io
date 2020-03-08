---
title:  LeetCode每日一题（一）：用队列实现栈
tags:
- LeetCode
- 算法
date:   2020-03-01 20:00:00
categories: 数据结构与算法
---

摘要：种一棵树最好的时间是十年前，其次是现在。

这是我的微信公众号第一篇文章。以前一直是我想做，但我又一直没有做。一方面是我比较懒...另一方面是我还没有信心去做好这件事。“如果没有去尝试，又怎会觉得自己的能力不够？干脆就把它当做一个记录，一个愿意把自己的事情去分享的笔记本。即使没有人去关注，也要持续的输出下去。”这就是说服我的理由。

所以，我决定今天把这颗种子种在这个片土地里，等待着它生根发芽，破土而出。

## LeetCode每日一题

春招在即，正在刷 OJ 的我碰巧看见了 **LeetCode** 上的这个活动。碰巧，这个活动从今天开始的。不多BB，开刷！

**225.用队列实现栈**

使用队列实现栈的下列操作：

* push(x) -- 元素 x 入栈
* pop() -- 移除栈顶元素
* top() -- 获取栈顶元素
* empty() -- 返回栈是否为空

## Solve

学过数据结构的基本知识我们可以得知：

* 队列：先进先出
* 栈：先进后出

只要将队列的队头指针作为栈的栈顶指针即可，没有什么难度。 C语言实现：

```c
#define MaxSize 50

typedef struct { // 定义结构体
    int data[MaxSize];
    int front, rear;
} MyStack;

/** Initialize your data structure here. */

MyStack* myStackCreate() {
    MyStack *obj = (MyStack*)malloc(sizeof(MyStack));
    obj -> front = obj -> rear = -1;
    return obj;
}

/** Push element x onto stack. */
void myStackPush(MyStack* obj, int x) {
    obj -> rear ++;
    obj -> data[obj -> rear] = x;
}

/** Removes the element on top of the stack and returns that element. */
int myStackPop(MyStack* obj) {
    int x = obj -> data[obj -> rear];
    obj -> rear --;
    return x;
}

/** Get the top element. */
int myStackTop(MyStack* obj) {
    return obj -> data[obj -> rear];
}

/** Returns whether the stack is empty. */
bool myStackEmpty(MyStack* obj) {
    return (obj -> front == obj -> rear);
}

void myStackFree(MyStack* obj) {
    free(obj);
}
```

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://github.com/RunCoderHang/LeetCode-Notes/blob/master/image/wxgzh-hang.png">
</div>