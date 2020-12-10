---
title: 剑指offer-链表中环的入口结点
date: 2020-03-24 22:40:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[链表中环的入口结点](https://www.nowcoder.com/practice/253d2c59ec3e4bc68da16833f79a38e4?tpId=13&tqId=11208&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述

给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。

<!--more-->

### 思路:

 1. 第一步,找是否有环,这里设置快慢指针,快指针一次两步,慢指针一次一步,若题两个指针相遇,则表明该链表有环

    证明若存在环,则快慢指针必定相遇

    	1. 若慢指针比快指针快一步,则前进一次,两者相遇
     	2. 若慢指针比快指针快2步,则前进一次,两者相差一步,回到问题1
     	3. 对于N,若慢指针比快指针快N步,则前进一次,两者相差N-1步
     	4. 综上,两者必定相遇

 2. 第二步,确定环的入口

    我们假设起始点到环入口距离为A,快慢指针相遇位置距离环入口为B,那么环剩余长度为C

    网上有证明是快指针比慢指针恰好多走一圈.不过这个应该是要在满足A>=(B+C)的情况的,很容易举例,假设A为5,B+C=2,很明显快指针会多走很多圈的

    因此我们这样考虑,假设相遇时快指针已经在环中走了N圈,那么

    ​		2(A+B)=A+B+(B+C)*N

    ​	=>	A=(B+C)*(N-1)+C

    因此我们不难发现,如果设置两个指针,一个指针从起始指针出发,一个指针从相遇点出发,每次两者都前进一步,当这两个指针相遇时,相遇点就是环的入口

### code

```java
public ListNode EntryNodeOfLoop(ListNode pHead)
{
    ListNode fast=pHead;
    ListNode slow=pHead;
    ListNode start=null;
    boolean flag=false;
    while(fast.next!=null&&fast.next.next!=null) {
        fast=fast.next.next;
        slow=slow.next;
        if(fast==slow) {
            flag=true;
            break;
        }
    }
    if(flag) {
        start=pHead;
        while(slow!=start) {
            slow=slow.next;
            start=start.next;
        }
    }
    return start;
}
```



