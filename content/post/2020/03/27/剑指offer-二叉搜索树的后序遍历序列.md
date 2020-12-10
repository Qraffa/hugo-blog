---
title: 剑指offer-二叉搜索树的后序遍历序列
date: 2020-03-27 14:49:07
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[二叉搜索树的后序遍历序列](https://www.nowcoder.com/practice/a861533d45854474ac791d90e447bafd?tpId=13&tqId=11176&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。

<!--more-->
### 思路:

​	根据二叉搜索树的性质,左子树都比根小,右子树都比根大.

​	首先找到根结点,也就是最后一位,然后再找到左右子树的分隔处,然后递归判断左右子树

​	存在一个疑问,数组为空,到底是true还是false?我觉得应该是true

​	但在剑指offer给出的代码上,首先判断数组长度,若为0则返回false.但是在后面递归找子树的时候,却又判断了子数组的大小,若<=0,则返回true. 确实有点看不懂这操作

### code

```java
public boolean VerifySquenceOfBST(int [] sequence) {
    if(sequence==null || sequence.length==0) {
        return false;
    }
    if(sequence.length==1) {
        return true;
    }
    int len=sequence.length;
    int root=sequence[len-1];
    int pos=0;
    for(;pos<len-1;++pos) {
        if(sequence[pos]>root) {
            break;
        }
    }
    int r=pos;
    for(;r<len-1;++r) {
        if(sequence[r]<root) {
            return false;
        }
    }
    return pos>0?VerifySquenceOfBST(Arrays.copyOfRange(sequence,0,pos)):true
        && (len-1-pos)>0?VerifySquenceOfBST(Arrays.copyOfRange(sequence,pos,len-1)):true;
}
```

