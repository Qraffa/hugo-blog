---
title: 剑指offer-数组中只出现一次的数字
date: 2020-05-03 15:11:58
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[数组中只出现一次的数字](https://www.nowcoder.com/practice/e02fdb54d7524710a7d664d082bb7811?tpId=13&tqId=11193&tPage=3&rp=3&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述

一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。

时间复杂度O(N),空间复杂度O(1)

<!--more-->
### 思路

剑指offer思路.

首先将数组中的数组做异或和,这个结果是两个数字的异或和.

然后找到这个异或和的最低位的1的位置,记为pos(这个异或和必定不为0,一定存在至少一个1)

然后再遍历数组,对每一位数字,如果它的第pos位为1,那么就用res1做异或和,反之用res2做异或和.

因为对于两个只出现一次的数字,他们在pos位一定不相同.那么就会分别分配到res1,res2

而对于那些出现两次的数字,无论他们的pos位如何,一定会在同一个res做两次异或.

最后,res1和res2就是两个只出现一次的数字.

### code

```java
public void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {
    int sum = 0;
    int len = array.length;
    for (int i = 0; i < len; i++) {
        sum ^= array[i];
    }
    int tmps = sum;
    int cnt = 0;
    while((tmps & 1) != 1) {
        tmps >>>= 1;
        cnt++;
    }

    int res1=0,res2=0;
    for (int i = 0; i < len; i++) {
        if(yi(array[i],cnt) == 1) {
            res1 ^= array[i];
        } else {
            res2 ^= array[i];
        }
    }
    num1[0] = res1;
    num2[0] = res2;
}

public int yi(int n,int cnt) {
    n >>>= cnt;
    return n & 1;
}
```

