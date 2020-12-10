---
title: "Alg 完全平方数"
date: 2020-11-25T17:13:15+08:00
tags:
- 算法
categories: 
- 算法
---

## 完全平方数

### 题目描述

给定正整数 *n*，找到若干个完全平方数（比如 `1, 4, 9, 16, ...`）使得它们的和等于 *n*。你需要让组成和的完全平方数的个数最少。

<!--more-->

### 思路

最开始写了个递归dp，超时了。

这里可以用[拉格朗日四平方和定理](https://blog.csdn.net/l_mark/article/details/89044137)

> **定理内容：**
> 每个正整数均可表示成不超过四个整数的平方之和
>
> **重要的推论：**
>
> 1. 数 n 如果只能表示成四个整数的平方和，不能表示成更少的数的平方之和，必定满足4^a(8b+7)
> 2. 如果 n%4==0，k=n/4，n 和 k 可由相同个数的整数表示

因此，

1. 先判断是否是4个整数组成，即验证4^a(8b+7)，满足则结果为4
2. 再判断是否为完全平方数，满足则结果为1
3. 再判断能否由两个数的平方和组成，这里我使用的枚举的方法，满足则结果为2
4. 以上都不符合，则结果为3

### code

```go
func numSquares(n int) int {
	tn := n
	for tn%4 == 0 {
		tn /= 4
	}
	if tn%8 == 7 {
		return 4
	}
	sq := (int)(math.Sqrt(float64(n)))
	if sq*sq == n {
		return 1
	}
	for i := sq; i >= 1; i-- {
		sqs := n - i*i
		sqsq := (int)(math.Sqrt(float64(sqs)))
		if sqsq*sqsq+i*i == n {
			return 2
		}
	}
	return 3
}
```

