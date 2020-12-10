---
title: 剑指offer-对称的二叉树
date: 2020-03-26 00:00:05
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[对称的二叉树](https://www.nowcoder.com/practice/ff05d44dfdb04e1d83bdbdab320efbcb?tpId=13&tqId=11211&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。

<!--more-->
### 思路:

​	题目说`如果一个二叉树同此二叉树的镜像是同样的，定义其为对称`,因此考虑对左子树做个镜像,然后与右子树比较各个结点即可

​	注意判断空指针

### code

```java
boolean isSymmetrical(TreeNode pRoot)
{
    if(pRoot==null || pRoot.left==null && pRoot.right==null) {
        return true;
    }
    if(pRoot.left==null || pRoot.right==null) {
        return false;
    }
    Mirror(pRoot.left);
    return comNode(pRoot.left,pRoot.right);
}

boolean comNode(TreeNode left,TreeNode right) {
    if(left==null&&right==null) {
        return true;
    }
    if(left!=null&&right!=null) {
        if(left.val==right.val) {
            return comNode(left.left,right.left)&&comNode(left.right,right.right);
        }
    }
    return false;
}

void Mirror(TreeNode root) {
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

