---
title:  LeetCode每日一题（二）：反转链表
tags:
- LeetCode
- 算法
date:   2020-03-02 20:00:00
categories: 数据结构与算法
---

以前考研时候做过这种题目，可以使用不同中方法反转。空间复杂度也可以保持为： **O(1)**

## 一、题目

反转一个单链表。

**示例：**

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

## 二、思考

#### 2.1迭代

**逆置法：**

使用三个指针（加上头指针），作用于不同的节点。从开始节点到尾结点一个一个地逆置指针方向。

**头插法：**

根据创建单链表使用的头插法。首先需要创建一个没有数据的头指针，之后相当于创建单链表一样，利用头插法原理将每个节点插入链表。

* 时间复杂度：O(n)
* 空间复杂度：O(1)

#### 2.2递归

通过递归，可以从最后一个节点，依次改变指针方向，逆置单链表。

* 时间复杂度：O(n)
* 空间复杂度：O(n) 

## 三、代码实现

**逆置法**

```c
// Reverse list
struct ListNode* reverseList(struct ListNode* head){
    struct ListNode *p, *r;
    p = head -> next;
    head -> next = NULL;
    while(p != NULL) {
        r = p -> next;
        p -> next = head;
        head = p;
        p = r;
    }
    return head;
}
```

**头插法**

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode *p, *q;
        ListNode *L = new ListNode(1);  
        p = head;
        L -> next = NULL;
        while(p) {
            q = p;
            p = p -> next;
            q -> next= L -> next;
            L -> next = q;
        }
        head = q;
        return head;
    }
};
```

**递归**

```c
struct ListNode* reverseList(struct ListNode* head) {
    if(head == NULL || head -> next == NULL)  return head;

    struct ListNode* s = reverseList(head->next); 
    head -> next -> next = head; 
    head -> next = NULL;

    return s;
}
```

<div align="center">
    <hr style="height:1px;"/>
    <br>
    <img width="200px" src="https://runcoderhang.github.io/thumbnails/wxgzh-hang.png">
</div>