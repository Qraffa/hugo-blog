---
title: 剑指offer-复杂链表的复制
date: 2020-03-27 14:49:07
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[复杂链表的复制](https://www.nowcoder.com/practice/f836b2c43afc4b35ad6adc41ec941dba?tpId=13&tqId=11178&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

<!--more-->
### 思路:

​	暴力和hash的做法就不说了,这里主要实现一下剑指offer书上给出的第三种优化

 1. 将链表各结点复制一份

    例如`1->2->3->4->5`变成`1->1'->2->2'->3-3'->4->4'->5->5'`

	2. 修改复制出来的结点的random指向

	3. 将这链表拆成两个链表

主要注意null的问题

### code

```java
public static RandomListNode Clone(RandomListNode pHead)
{
    if(pHead==null) {
        return null;
    }
    RandomListNode head=pHead;
    while(head!=null) {
        RandomListNode node=new RandomListNode(head.label);
        node.next=head.next;
        head.next=node;
        head=node.next;
    }
    head=pHead;
    while(head!=null) {
        head.next.random=head.random==null?null:head.random.next;
        head=head.next.next;
    }
    head=pHead;
    RandomListNode res,chead;
    res=chead=head.next;
    while(head!=null) {
        head.next=chead.next;
        head=head.next;
        chead.next=head==null?null:head.next;
        chead=chead.next;
    }
    return res;
}
```

