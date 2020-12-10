---
title: 剑指offer-青蛙跳台阶问题
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

### 题目描述:

一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

​	没啥好说的

### code

```java
public int numWays(int n) {
    int mod=1000000007;
    int[] arr=new int[105];
    arr[0]=1;
    arr[1]=1;
    arr[2]=2;
    for(int i=3;i<=100;++i) {
        arr[i]=(arr[i-1]%mod+arr[i-2]%mod)%mod;
    }
    return arr[n];
}
```

