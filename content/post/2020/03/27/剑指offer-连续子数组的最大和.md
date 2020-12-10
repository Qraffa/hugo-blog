---
title: 剑指offer-连续子数组的最大和
date: 2020-03-27 14:49:07
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[连续子数组的最大和](https://www.nowcoder.com/practice/459bd355da1549fa8a49e350bf3df484?tpId=13&tqId=11183&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

HZ偶尔会拿些专业问题来忽悠那些非计算机专业的同学。今天测试组开完会后,他又发话了:在古老的一维模式识别中,常常需要计算连续子向量的最大和,当向量全为正数的时候,问题很好解决。但是,如果向量中包含负数,是否应该包含某个负数,并期望旁边的正数会弥补它呢？例如:{6,-3,-2,7,-15,1,2,2},连续子向量的最大和为8(从第0个开始,到第3个为止)。给一个数组，返回它的最大连续子序列的和，你会不会被他忽悠住？(子向量的长度至少是1)

<!--more-->
### 思路:

​	首先定义数组ans,ans[i]表示以第i项结尾的子数组的最大和

​	对于第i个数字,ans[i-1]+arr[i]>arr[i],则ans[i]=ans[i-1]+arr[i].否则ans[i]=arr[i]

### code

```java
public int FindGreatestSumOfSubArray(int[] array) {
    int len=array.length;
    int[] ans=new int[len];
    ans[0]=array[0];
    for (int i = 1; i < len; i++) {
        if(ans[i-1]+array[i]>array[i]) {
            ans[i]=ans[i-1]+array[i];
        } else {
            ans[i]=array[i];
        }
    }
    int maxx=ans[0];
    for (int i = 1; i < len; i++) {
        maxx=Math.max(ans[i],maxx);
    }
    return maxx;
}
```

