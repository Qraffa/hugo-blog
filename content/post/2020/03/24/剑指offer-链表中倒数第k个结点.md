---
title: 剑指offer-链表中倒数第k个结点
date: 2020-03-24 22:40:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[链表中倒数第k个结点](https://www.nowcoder.com/practice/529d3ae5a407492994ad2a246518148a?tpId=13&tqId=11167&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

输入一个链表，输出该链表中倒数第k个结点。

<!--more-->

### 思路:

​	假设以1为起始下标,链表长度为N,那么倒数第K个结点所在的位置是N-(K-1)

​	但由于不知道N的大小,因此设置前后指针,先让前指针走K-1步,这样前后指针的距离是K-1,再让前后指针一起移动,当前指针移动到链表尾时,后指针所在的位置就是倒数第K个结点.

### code

```java
public ListNode FindKthToTail(ListNode head,int k) {
    if(head==null || k==0) {
        return null;
    }
    ListNode pre=head;
    ListNode ans=head;
    k--;
    while(pre.next!=null&& k>0 ) {
        pre=pre.next;
        k--;
    }
    if(k!=0) {
        return null;
    }
    while(pre.next!=null) {
        pre=pre.next;
        ans=ans.next;
    }
    return ans;
}
```

