---
title: 剑指offer-从尾到头打印链表
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof)

### 题目描述

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

<!--more-->

### 思路:

​	没啥好说的

### code

```java
public int[] reversePrint(ListNode head) {
    int cnt=0;
    Stack<Integer> stack=new Stack<>();
    while(head!=null) {
        stack.push(head.val);
        head=head.next;
        cnt++;
    }
    int[] arr=new int[cnt];
    int i=0;
    while(!stack.empty()) {
        arr[i++]=stack.pop();
    }
    return arr;
}
```
