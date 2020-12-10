---
title: 剑指offer-二叉树的镜像
date: 2020-03-26 00:00:05
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[二叉树的镜像](https://www.nowcoder.com/practice/564f4c26aa584921bc75623e48ca3011?tpId=13&tqId=11171&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

操作给定的二叉树，将其变换为源二叉树的镜像。

<!--more-->

## 输入描述:

```sh
二叉树的镜像定义：源二叉树 
    	    8
    	   /  \
    	  6   10
    	 / \  / \
    	5  7 9 11
    	镜像二叉树
    	    8
    	   /  \
    	  10   6
    	 / \  / \
    	11 9 7  5
```

### 思路:

​	直接递归交换左右子树即可

### code

```java
public void Mirror(TreeNode root) {
    if(root==null) {
        return;
    }
    TreeNode node=root.left;
    root.left=root.right;
    root.right=node;
    Mirror(root.left);
    Mirror(root.right);
}
```

