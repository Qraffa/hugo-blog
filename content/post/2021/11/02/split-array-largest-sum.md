---
title: "分割数组的最大值"
date: 2021-11-02T00:00:03+08:00
url: post/2021/11/02/split-array-largest-sum
tags:
- 算法
categories: 
- 算法
---

## 分割数组的最大值

### 题目描述

给定一个非负整数数组 nums 和一个整数 m ，你需要将这个数组分成 m 个非空的连续子数组。

设计一个算法使得这 m 个子数组各自和的最大值最小。

 

示例 1：

输入：nums = [7,2,5,10,8], m = 2
输出：18
解释：
一共有四种方法将 nums 分割为 2 个子数组。 其中最好的方式是将其分为 [7,2,5] 和 [10,8] 。
因为此时这两个子数组各自的和的最大值为18，在所有情况中最小。
示例 2：

输入：nums = [1,2,3,4,5], m = 2
输出：9
示例 3：

输入：nums = [1,4,4], m = 3
输出：4

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/split-array-largest-sum
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 思路

1. 对数组维护前缀和
2. 二分枚举答案，对于每个答案，二分查找区间和划分验证

### code

```go
func splitArray(nums []int, m int) int {
  // 特殊处理
	if len(nums) == 1 {
		return nums[0]
	}
	var l, r int
	var ans int = 0
  // 维护前缀和数组
	sum := make([]int, len(nums)+1)
	for i := 0; i < len(nums); i++ {
		l = max(l, nums[i])
		r += nums[i]
		sum[i+1] = sum[i] + nums[i]
	}
  // 特殊处理
	if m == 1 {
		return sum[len(sum)-1]
	}
	ans = sum[len(sum)-1]
  // 二分枚举答案
	for l < r {
		mid := l + (r-l)/2
		k, last := 0, 0
		ok := true
		isans := true
		for k < m {
      // 二分查找区间和划分
			p := sort.Search(len(sum)-last, func(i int) bool {
				return sum[last+i]-sum[last] > mid
			}) + last
			k++
			last = p - 1
      // 答案偏小，最后剩余的部分大于答案，不满足，需要放大答案
			if k+1 == m && sum[len(sum)-1]-sum[last] > mid {
				isans = false
				ok = true
				break
			}
      // 划分次数少于m，已经完成，表示答案符合，但偏大，需要缩小答案
      // 划分次数少于m，表示可以在其他区间继续划分，依然满足条件，所以答案是符合的
			if k < m-1 && last >= len(sum)-1 {
				ok = false
				isans = true
				break
			}
		}
		if isans {
			ans = min(ans, mid)
			r = mid
		} else {
			if ok {
				l = mid + 1
			} else {
				r = mid
			}
		}
	}
	return ans
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}
func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```

