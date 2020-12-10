---
title: 剑指offer-数值的整数次方
date: 2020-03-24 16:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

### 题目描述:

实现函数double Power(double base, int exponent)，求base的exponent次方。不得使用库函数，同时不需要考虑大数问题。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

​	快速幂

​	注意点:n要先转为long类型,在这WA了一次

### code

```java
public double myPow(double x, int n) {
    long pow=n;
    if(pow<0) {
        x=1/x;
        pow=-pow;
    }
    double ans=1;
    while(pow>0) {
        if(pow%2==1) {
            ans=x*ans;
        }
        pow/=2;
        x=x*x;
    }
    return ans;
}
```

