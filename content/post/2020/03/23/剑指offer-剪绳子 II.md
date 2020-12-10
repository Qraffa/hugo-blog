---
title: 剑指offer-剪绳子 II
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[剪绳子 II](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof)

### 题目描述:

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m] 。请问 k[0]*k[1]*...*k[m] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

​	[剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof)加强版

​	因为数据比较大,考虑贪心+快速幂

​	不难发现,大于3的绳子都是切成3和2的乘积划算,同时应该尽可能多地拆成3,再把剩余的拆成2

​	若n%3==0,则全部切为3

​	若n%3==1,则把一个3和1组成4,切为2*2

​	若n%3==2,则切成n/3个3和1个2

​	做快速幂注意中间变量溢出

### code

```java
public int cuttingRope(int n) {
    if(n==2||n==3) {
        return n-1;
    }
    if(n==1||n==4) {
        return n;
    }
    int pow=n/3;
    int pow2=n%3;
    int mod=1000000007;
    if(pow2==1){
        pow--;
        pow2++;
    } else if(pow2==2) {
        pow2--;
    }
    return (int)(quick_pow(3,pow,mod)*(long)Math.pow(2,pow2)%mod);
}
int quick_pow(long a,long b,long c) {
    long ans=1;
    a=a%c;
    while(b>0) {
        if(b%2==1) {
            ans=(ans*a)%c;
        }
        b/=2;
        a=(a*a)%c;
    }
    return (int)ans;
}
```

