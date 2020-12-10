---
title: 剑指offer-斐波那契数列
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof) 

### 题目描述:

写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项。斐波那契数列的定义如下：

F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

​	没啥好说的

### code

```java
public int fib(int n) {
    int mod=1000000007;
    int[] arr=new int[105];
    arr[1]=1;
    arr[2]=1;
    for(int i=3;i<=100;++i) {
        arr[i]=(arr[i-1]%mod+arr[i-2]%mod)%mod;
    }
    return arr[n];
}
```

