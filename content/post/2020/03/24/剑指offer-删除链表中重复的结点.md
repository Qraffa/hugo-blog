---
title: 剑指offer-删除链表中重复的结点
date: 2020-03-24 22:40:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[删除链表中重复的结点](https://www.nowcoder.com/practice/fc533c45b73a41b0b44ccba763f866ef?tpId=13&tqId=11209&tPage=3&rp=3&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表1->2->3->3->4->4->5 处理后为 1->2->5

<!--more-->

### 思路:

​	模拟,就是操作有点复杂,需要细心

​	主要思路是设置前后指针一起移动

​	如果发现cur指针处出现重复,那么一直遍历下去找到未重复的结点,重新链接.

​	难点在于头部的重复处理.这里选择再添加一个头指针,需要注意的是对于pre指针和cur指针相同,需要特殊处理

​	举例一:`1 1 2` 此时,pre在第一个1,cur在第二个1

​	如果这个头没有处理,最后还是`1 1 2`

​	举例二:`1 1 1 2` 此时,pre在第一个1,cur在第二个1

​	如果这个头没有处理,最后是`1 2`	

​	因此这里的做法是若pre和cur相同,则在cur后在添加一个相同的结点,避免出现举例一的情况.

### code

```java
public ListNode deleteDuplication(ListNode pHead)
{
    ListNode pre=pHead;
    if(pre==null) {
        return pHead;
    }
    ListNode cur=pHead.next;
    if(cur==null) {
        return pHead;
    }
    ListNode head=new ListNode(-1);
    head.next=pre;
    boolean flag=false;
    if(pre.val==cur.val) {
        flag=true;
        ListNode newNode=new ListNode(pre.val);
        newNode.next=cur.next;
        cur.next=newNode;
    }
    while(cur!=null) {
        if(cur.next!=null&&cur.val==cur.next.val) {
            while(cur.next!=null&&cur.val==cur.next.val) {
                cur=cur.next;
            }
            cur=cur.next;
            pre.next=cur;
        } else {
            pre=pre.next;
            cur=cur.next;
        }
    }
    head=head.next;
    if(flag) {
        head=head.next;
    }
    return head;
}
```

