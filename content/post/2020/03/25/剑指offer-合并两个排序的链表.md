---
title: 剑指offer-合并两个排序的链表
date: 2020-03-25 01:53:09
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[合并两个排序的链表](https://www.nowcoder.com/practice/d8b6b4358f774294a89de2a6ac4d9337?tpId=13&tqId=11169&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

<!--more-->
### 思路:

​	设置个头指针,遍历两链表,谁小那头指针的next就指向谁,最后把剩下的链上

### code

```java
public ListNode Merge(ListNode list1,ListNode list2) {
    ListNode head=new ListNode(-1);
    ListNode ans=head;
    ListNode p1,p2;
    p1=list1;
    p2=list2;
    while(p1!=null && p2!=null) {
        if(p1.val<p2.val) {
            head.next=p1;
            p1=p1.next;
            head=head.next;
        } else {
            head.next=p2;
            p2=p2.next;
            head=head.next;
        }
    }
    if(p1==null) {
        head.next=p2;
    } else {
        head.next=p1;
    }
    return ans.next;
}
```

