---
title: 剑指offer-反转链表
date: 2020-03-25 01:53:09
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[反转链表](https://www.nowcoder.com/practice/75e878df47f24fdc9dc3e400ec6058ca?tpId=13&tqId=11168&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

输入一个链表，反转链表后，输出新链表的表头。

<!--more-->
### 思路:

​	设置前置指针pre,现指针cur,下一指针nex

	1. 将cur的next指向pre
 	2. pre指向cur
 	3. cur指向nex
 	4. nex指向nex的next

### code

```java
public ListNode ReverseList(ListNode head) {
    if(head==null){
        return null;
    }
    ListNode pre,cur,nex;
    pre=null;
    cur=head;
    nex=head.next;
    while(cur!=null) {
        cur.next=pre;
        pre=cur;
        cur=nex;
        if(cur==null) {
            break;
        }
        nex=nex.next;
    }
    return pre;
}
```

