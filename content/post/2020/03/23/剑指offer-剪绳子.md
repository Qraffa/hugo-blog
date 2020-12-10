---
title: 剑指offer-剪绳子
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof)

### 题目描述:

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m] 。请问 k[0]*k[1]*...*k[m] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/jian-sheng-zi-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

​	简单dp

​	注意初始状态dp[2]=1,dp[3]=2

​	状态转移方程:

​	`dp[i]=Math.max(dp[i],Math.max(dp[j],j)*Math.max(dp[i-j],i-j));`

​	与自身比较是2和3的特殊情况,2和3在后面的切割状态应该是不切是最佳的.

​	~~当然这题打表也是可以的~~

### code

```java
public int cuttingRope(int n) {
    int[] dp=new int[60];
    dp[1]=1;
    dp[2]=1;
    dp[3]=2;
    for(int i=4;i<=n;++i) {
        dp[i]=0;
    }
    for(int i=4;i<=n;++i) {
        for(int j=1;j<=i/2;++j) {
            dp[i]=Math.max(dp[i],Math.max(dp[j],j)*Math.max(dp[i-j],i-j));
        }
    }
    return dp[n];
}
```

