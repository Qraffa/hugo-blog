---
title: 剑指offer-重建二叉树
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof) 

### 题目描述

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

<!--more-->

### 思路:

​	直接递归建树

​	根据前序找到根节点,根据该根节点从中序前半为左子树的中序,后半为右子树的中序

​	根据左子树的中序的数量确定前序中左子树的前序,剩余为右子树的前序

### code

```java
public TreeNode buildTree(int[] preorder, int[] inorder) {
    if(preorder.length==0) {
        return null;
    }
    TreeNode node=new TreeNode(preorder[0]);
    int len=inorder.length;
    int rootpos=0;
    int[] leftInArr;
    int[] leftPreArr;
    for(int i=0;i<len;++i) {
        if(inorder[i]==preorder[0]) {
            rootpos=i;
            break;
        }
    }
    leftInArr=Arrays.copyOfRange(inorder,0,rootpos);
    leftPreArr=Arrays.copyOfRange(preorder,1,leftInArr.length+1);
    node.left=buildTree(leftPreArr,leftInArr);

    int[] rightInArr=Arrays.copyOfRange(inorder,rootpos+1,len);
    int[] rightPreArr=Arrays.copyOfRange(preorder,leftInArr.length+1,len);
    node.right=buildTree(rightPreArr,rightInArr);
    return node;
}
```

