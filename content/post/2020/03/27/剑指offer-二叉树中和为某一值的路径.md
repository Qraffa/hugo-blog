---
title: 剑指offer-二叉树中和为某一值的路径
date: 2020-03-27 14:49:07
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[二叉树中和为某一值的路径](https://www.nowcoder.com/practice/b736e784e3e34731af99065031301bca?tpId=13&tqId=11177&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

输入一颗二叉树的根节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。(注意: 在返回值的list中，数组长度大的数组靠前)

<!--more-->
### 思路:

​	考虑使用递归,类似前序遍历.使用栈来存储目前走过的路径.

### code

```java
Stack<Integer> stack;
ArrayList<ArrayList<Integer>> lists;

public ArrayList<ArrayList<Integer>> FindPath(TreeNode root, int target) {
    stack=new Stack<>();
    lists=new ArrayList<>();
    if(root==null) {
        return lists;
    }
    getPath(root,target);
    lists.sort(new Comparator<ArrayList<Integer>>() {
        @Override
        public int compare(ArrayList<Integer> integers, ArrayList<Integer> t1) {
            return integers.size()<t1.size()?1:-1;
        }
    });
    return lists;
}

void getPath(TreeNode root,int target) {
    if(root.val>target) {
        return;
    }
    stack.push(root.val);
    if(root.val==target && root.left==null && root.right==null) {
        ArrayList<Integer> list=new ArrayList<>();
        while(!stack.empty()) {
            list.add(0,stack.peek());
            stack.pop();
        }
        lists.add(list);
        int len=list.size();
        for (int i = 0; i < len; i++) {
            stack.push(list.get(i));
        }
        stack.pop();
        return;
    }
    if(root.left!=null){
        getPath(root.left,target-root.val);
    }
    if(root.right!=null){
        getPath(root.right,target-root.val);
    }
    stack.pop();
}
```

