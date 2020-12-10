---
title: 剑指offer-二进制中1的个数
date: 2020-03-24 16:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

### 题目描述:

请实现一个函数，输入一个整数，输出该数二进制表示中 1 的个数。例如，把 9 表示成二进制是 1001，有 2 位是 1。因此，如果输入 9，则该函数输出 2。

示例 1：

输入：00000000000000000000000000001011
输出：3
解释：输入的二进制串 00000000000000000000000000001011 中，共有三位为 '1'。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

​	我不得不吐槽下leetcode的这个样例,如样例一,写的输入是1011,实际函数中的参数传的是1011的十进制也就是11

​	两种思路:

 1. 按位与,每次n与1与,得到的是最后一位的1/0,然后n无符号右移

 2. leetcode大神的思路:`n&(n-1)`

    为什么是可行的呢,尝试就会发现n-1是把n的最后一位变为0,其后变为1,例如

    n	->	101100

    n-1->	101011

    因此,这样每次都能直接找到1的位置,统计做了多少次即可

### code

```java
// 解法一
public int hammingWeight(int n) {
    int cnt=0;
    // 这里必须是n!=0,不能是n>0
    while(n!=0) {
        cnt+=n&1;
        n>>>=1;
    }
    return cnt;
}
```

```java
// 解法二
public int hammingWeight(int n) {
    int cnt=0;
    // 这里必须是n!=0,不能是n>0
    while(n!=0) {
        cnt++;
        n=n&(n-1);
    }
    return cnt;
}
```

