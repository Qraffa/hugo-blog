---
title: 剑指offer-树的子结构
date: 2020-03-26 00:04:26
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[树的子结构](https://www.nowcoder.com/practice/6e196c44c7004d15b1610b9afca8bd88?tpId=13&tqId=11170&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）

<!--more-->
### 思路:

 1. 找到合适的根结点作为入口,即根结点的值相同.二叉树的前序递归遍历即可

 2. 若找到合适的根节点,则递归遍历他们的左右子树,必须要左右子树都满足才算匹配.

    如果查询树的结点为null,则返回true.

    如果查询树的结点不为null,并且匹配树为null,则说明不匹配,返回false

### code

```java
boolean flag=false;

public boolean HasSubtree(TreeNode root1,TreeNode root2) {
    if(flag) {
        return true;
    }
    if(root1!=null && root2!=null) {
        if(root1.val==root2.val) {
            flag=getSub(root1,root2);
        }
        flag=HasSubtree(root1.left,root2)||HasSubtree(root1.right,root2);
    }
    return flag;
}

boolean getSub(TreeNode pattern,TreeNode node) {
    if(node==null) {
        return true;
    }
    if( (node!=null && pattern==null) || node.val!=pattern.val) {
        return false;
    }
    return getSub(pattern.left,node.left)&&getSub(pattern.right,node.right);
}
```

