---
title: "Alg 除自身以外数组的乘积"
date: 2020-11-24T15:58:51+08:00
tags:
- 算法
categories:
- 算法
---

## 除自身以外数组的乘积

### 题目描述

给你一个长度为 *n* 的整数数组 `nums`，其中 *n* > 1，返回输出数组 `output` ，其中 `output[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积。

<!--more-->

**示例:**

```
输入: [1,2,3,4]
输出: [24,12,8,6]
```

**提示：**题目数据保证数组之中任意元素的全部前缀元素和后缀（甚至是整个数组）的乘积都在 32 位整数范围内。

**说明:** 请**不要使用除法，**且在 O(*n*) 时间复杂度内完成此题。

**进阶：**
你可以在常数空间复杂度内完成这个题目吗？（ 出于对空间复杂度分析的目的，输出数组**不被视为**额外空间。）

### 思路

首先需要排除掉求整个数组乘积再除以对应数的方法，当数组中有0时不可用。

#### 思路一：

前缀和思路，维护left数组，表示前缀积，维护right数组，表示后缀积。

对于[1,2,3,4]，left数组[1,1,2,6]，right数组[24,12,4,1]

因此结果ans[i]=left[i]*right[i]

#### code

```go
func productExceptSelf(nums []int) []int {
	ans, r := make([]int, len(nums)), make([]int, len(nums))
	ans[0] = 1
	for i := 1; i < len(ans); i++ {
		ans[i] = ans[i-1] * nums[i-1]
	}
	r[len(nums)-1] = 1
	for i := len(nums) - 2; i >= 0; i-- {
		r[i] = r[i+1] * nums[i+1]
	}
	for i := 0; i < len(nums)-1; i++ {
		ans[i] = ans[i] * r[i]
	}
	return ans
}
```

#### 思路二：

思路一的left/right数组其中一个只需要使用一次，因此可以使用一个变量来保存即可

维护left数组，表示前缀积，维护变量r，表示后缀积，从后往前遍历，维护更新left数组和后缀积r

```go
func productExceptSelf(nums []int) []int {
	ans := make([]int, len(nums))
	ans[0] = 1
	for i := 1; i < len(ans); i++ {
		ans[i] = ans[i-1] * nums[i-1]
	}
	r := nums[len(nums)-1]
	for i := len(ans) - 2; i >= 0; i-- {
		ans[i] = ans[i] * r
		r *= nums[i]
	}
	return ans
}
```

