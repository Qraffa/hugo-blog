---
title: 剑指offer-二叉搜索树的第k个结点
date: 2020-05-03 15:11:58
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[二叉搜索树的第k个结点](https://www.nowcoder.com/practice/ef068f602dde4d28aab2b210e859150a?tpId=13&tqId=11215&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述

给定一棵二叉搜索树，请找出其中的第k小的结点。例如， （5，3，7，2，4，6，8）  中，按结点数值大小顺序第三小结点的值为4。

<!--more-->

### 思路

二叉搜索树的中序遍历是非降序排序的。

因此可以考虑做中序遍历，然后遍历一个结点k--，当k为0时，就找到了第k个结点

### code

```java
int kk = 0;
TreeNode res = null;

void mid(TreeNode pRoot)
{
    // 中序遍历
    if(pRoot==null) {
        return;
    }
    mid(pRoot.left);
    kk--;
    if(kk == 0) {
        res =  pRoot;
    }
    mid(pRoot.right);
}

TreeNode KthNode(TreeNode pRoot, int k)
{
    kk = k;
    mid(pRoot);
    return res;
}
```

