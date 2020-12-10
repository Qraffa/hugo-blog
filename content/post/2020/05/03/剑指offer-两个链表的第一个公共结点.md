---
title: 剑指offer-两个链表的第一个公共结点
date: 2020-05-03 15:11:58
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[两个链表的第一个公共结点](https://www.nowcoder.com/practice/6ab1d9a29e88450685099d45c9e31e46?tpId=13&tqId=11189&tPage=3&rp=3&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述

输入两个链表，找出它们的第一个公共结点。（注意因为传入数据是链表，所以错误测试数据的提示是用其他方式显示的，保证传入数据是正确的）

<!--more-->
### 思路

理解题意.这里说的公共结点是只在其后的结点都是重合的.

大概就是个`Y`字形.

1->2->3->6->7

​		4->5↑

---

**思路一:**

​	首先在重合结点以后的结点都是相同的,那么我们可以考虑从后往前遍历,出现分岔点的地方就是公共结点.

​	但考虑到这是单向链表,无法进行从后往前遍历的.那么可以考虑用数据结构来保存链表结点,数组和栈都可以.

​	使用数组就是将所有结点存入数组,然后从后往前遍历.

​	使用栈就是将所有结点压栈,然后依次弹出.

​	这样我们可以得到从后往前遍历多少次,或者弹出栈多少次,然后对链表做一个求倒数第k个结点即可

**思路二:**

​	剑指offer思路.

​	还可以考虑从前往后遍历.

​	但考虑到两个链表在公共结点之前的长度可能不相同,那么需要先将两个链表的起点置于与公共结点相同的位置上.

​	先遍历两个链表,得到两个长度,将较长链表后移两个链表的长度差个位置.这样两个链表距离公共结点相同.

​	然后同时遍历两个链表,当出现两个结点相同时,就是公共结点.

### code

```java
// 思路一
public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
    Stack<Integer> stack1 = new Stack<>();
    Stack<Integer> stack2 = new Stack<>();
    ListNode p1 = pHead1;
    while(p1 != null) {
        stack1.push(p1.val);
        p1 = p1.next;
    }
    ListNode p2 = pHead2;
    while(p2 != null) {
        stack2.push(p2.val);
        p2 = p2.next;
    }
    int cnt = 0;
    while(!stack1.empty() && !stack2.empty()) {
        if(stack1.pop() == stack2.pop()) {
            cnt++;
        } else {
            break;
        }
    }
    return FindKthToTail(pHead1,cnt);
}

// 寻找链表倒数第k个结点
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



```java
// 思路二
public ListNode FindFirstCommonNode2(ListNode pHead1, ListNode pHead2) {
    int len1 = 0,len2 = 0;
    ListNode p1 = pHead1;
    while(p1 != null) {
        len1++;
        p1 = p1.next;
    }
    ListNode p2 = pHead2;
    while(p2 != null) {
        len2++;
        p2 = p2.next;
    }
    int sub = Math.abs(len1-len2);
    p1 = pHead1;
    p2 = pHead2;
    if(len1 > len2) while(sub-->0) p1=p1.next;
    else while(sub-->0) p2=p2.next;

    while(p1!=null && p2!=null) {
        if(p1 == p2) {
            return p1;
        } else {
            p1 = p1.next;
            p2 = p2.next;
        }
    }
    return null;
}
```

