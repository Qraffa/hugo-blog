---
title: "Alg 最长上升子序列"
date: 2020-11-26T19:01:13+08:00
tags:
- 算法
categories: 
- 算法
---

## 最长上升子序列

### 题目描述

给定一个无序的整数数组，找到其中最长上升子序列的长度。

**示例:**

输入: [10,9,2,5,3,7,101,18]
输出: 4 
解释: 最长的上升子序列是 [2,3,7,101]，它的长度是 4。

<!--more-->

### 思路

#### 思路一：

维护ans数组，ans[i]表示以nums[i]结尾的数组的最长上升子序列长度。时间复杂度n2

更新ans时，ans[i]=max(ans[j])+1 {其中0≤j<i，且nums[i]>nums[j]}

#### code

```go
func lengthOfLIS(nums []int) int {
	ans := make([]int, len(nums))
	res := 0
	for k, v := range nums {
		cnt := 0
		for j := k - 1; j >= 0; j-- {
			if v > nums[j] {
				cnt = max(cnt, ans[j])
			}
		}
		ans[k] = cnt + 1
		res = max(res, ans[k])
	}
	return res
}
func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```

#### 思路二：

维护ans数组，ans[i]表示长度为i的最长上升子序列的最小结尾数。可以保证ans数组是单调递增的。时间复杂度nlgn

对于nums[i]，二分查找ans中大于等于nums[i]的位置，更新ans，最终结果为ans数组长度。

#### code

```go
func lengthOfLIS(nums []int) int {
	ans := make([]int, 0)
	for _, v := range nums {
		pos := bs(ans, v)
		if pos > len(ans)-1 {
			ans = append(ans, 0)
		}
		ans[pos] = v
	}
	return len(ans)
}

func bs(nums []int, val int) int {
	l, r := 0, len(nums)
	for l < r {
		m := (r-l)/2 + l
		if nums[m] < val {
			l = m + 1
		} else {
			r = m
		}
	}
	return l
}
```

