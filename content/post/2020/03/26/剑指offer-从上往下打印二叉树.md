---
title: 剑指offer-从上往下打印二叉树
date: 2020-03-26 00:00:05
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[从上往下打印二叉树](https://www.nowcoder.com/practice/7fe2212963db4790b57431d9ed259701?tpId=13&tqId=11175&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

从上往下打印出二叉树的每个节点，同层节点从左至右打印。

<!--more-->
### 思路:

​	直接用队列做个层序遍历就行了

​	这牛客网天天就WA在空数据上面,emmm,一开始我判断root为null直接返回null,结果要返回空list.

### code

```java
public static ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
    ArrayList<Integer> list=new ArrayList<>();
    if(root==null) {
        return list;
    }
    Queue<TreeNode> queue=new LinkedList<>();
    queue.offer(root);
    while(!queue.isEmpty()) {
        list.add(queue.peek().val);
        if(queue.peek().left!=null) {
            queue.offer(queue.peek().left);
        }
        if(queue.peek().right!=null) {
            queue.offer(queue.peek().right);
        }
        queue.poll();
    }
    return list;
}
```

### 拓展一

​	剑指offer上提出新问题,要求分层打印二叉树.

<!-- more -->
### 思路:

​	要想办法知道当前层有多少结点要打印

​	因此,做法就是设置print标记当前层要打印的结点数量,每打印一个,print--,当print为0时,表示这层打印完

​	同时要设置next标记下一层结点个数,每插入一个子结点,next++

​	当print为0时,将next赋值print,next置0,开始下一层

### 拓展二

​	剑指offer上又提出新问题,要求之字形打印.即一层正序打印,一层逆序打印.

<!-- more -->
### 思路:

​	做法同拓展一,再添加一个新标记cnt,记录当前在第几层.当每次print为0时,cnt++

​	若当前层为奇数层,则正常打印

​	若当前层为偶数层,则先将需要打印的元素入栈,在该层结束时,做个出栈打印

​	