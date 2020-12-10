---
title: 剑指offer-二叉树的下一个结点
date: 2020-03-24 16:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[二叉树的下一个结点](https://www.nowcoder.com/practice/9023a0c988684a53960365b889ceaf5e?tpId=13&tqId=11210&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。

<!--more-->

### 思路:

​	写起来有点麻烦的一道题

​	对于一个结点,

	1. 若它有右子树,则下一个结点是它的右子树的最左端的结点
 	2. 若它没有右子树
      	1. 若该结点没有父结点,则下一个结点为null
      	2. 若该结点是它父结点的左孩子,则下一个结点是它的父节点
      	3. 若该结点是它父结点的右孩子,则向上寻找父结点
           	1. 若找到某一个结点,该结点是它父结点的左孩子,则下一个结点是该结点的父结点
           	2. 若没有找到,则下一个结点为null

### code

```java
public TreeLinkNode GetNext(TreeLinkNode pNode)
{
    TreeLinkNode ans=null;
    if(pNode.right!=null) {
        ans=pNode.right;
        while(ans.left!=null) {
            ans=ans.left;
        }
    } else {
        if(pNode.next==null) {
            ans=null;
        } else if(pNode==pNode.next.left){
            ans=pNode.next;
        } else {
            while(pNode.next!=null) {
                pNode=pNode.next;
                if(pNode.next!=null&&pNode.next.left==pNode){
                    ans=pNode.next;
                    break;
                }
            }
        }
    }
    return ans;
```

