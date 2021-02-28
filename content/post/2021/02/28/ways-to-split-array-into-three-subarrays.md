---
title: "将数组分成三个子数组的方案数"
date: 2021-02-28T23:16:33+08:00
url: ways-to-split-array-into-three-subarrays
tags:
- 算法
categories: 
- 算法
---

## 将数组分成三个子数组的方案数

### 题目描述

我们称一个分割整数数组的方案是 好的 ，当它满足：

- 数组被分成三个 非空 连续子数组，从左至右分别命名为 left ， mid ， right 。
- left 中元素和小于等于 mid 中元素和，mid 中元素和小于等于 right 中元素和。

给你一个 非负 整数数组 nums ，请你返回 好的 分割 nums 方案数目。由于答案可能会很大，请你将结果对 109 + 7 取余后返回。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/ways-to-split-array-into-three-subarrays
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

```
输入：nums = [1,2,2,2,5,0]
输出：3
解释：nums 总共有 3 种好的分割方案：
[1] [2] [2,2,5,0]
[1] [2,2] [2,5,0]
[1,2] [2,2] [5,0]
```

### 思路

数组被分成三部分，则需要两个分割点。两个点设为l,m，因此0<l<len-2，l<m<len-1

对数组做一个前缀和，

- 第一部分数组和为0~l，表示为Sl
- 第二部分数组和为l~m，表示为Sm-Sl
- 第三部分数组和为m~r，表示为Sr-Sm

因此根据题目，需要满足Sl≤Sm-Sl≤Sr-Sm

因此Sm≥2Sl，2Sm≤Sl+Sr

对于这题，首先枚举第一个分割点l，对于一个分割点l，尝试二分查找第二个分割点的上下边界

下边界找到第一个能使Sm≥2Sl成立的位置，上边界则表示找到第一个能使2Sm>Sl+Sr成立的位置

上下边界的距离即为该分割点l所能分隔的方案数，枚举所有可能的分割点l，总和即为答案

**说明：**

1. 在求下边界时，只考虑了满足左边不等式成立，因此是可能存在Sm≥2Sl，2Sm>Sl+Sr，即只满足左不等式，不满足右不等式。这种情况，会由于在求上边界时，因为要找第一个能使2Sm>Sl+Sr成立的位置，所以两个边界会是求在同一个位置的，因此该情况下，结果为0
2. 在求上边界时，只考虑了满足右边不等式成立，因此这里只要规定求上界时的范围是`下边界到数组长度`即可，这样由于位置大于下边界，那就已经满足有左不等式，虽然这里可能求的下边界存在左不等式也不满足的情况，但同样，这里上下边界会求在同一个位置，结果为0

### code

```go
func waysToSplit(nums []int) int {
	sum := make([]int, len(nums)+1)
	for i := 1; i <= len(nums); i++ {
		sum[i] = nums[i-1] + sum[i-1]
	}
	r := len(nums)
	var ans int
	for l := 1; l <= r-2; l++ {
		ml, mr := l+1, r
		left := sum[l]
		var m1, m2 int
		// lower
		m1 = sort.Search(mr-ml, func(i int) bool {
			return 2*left <= sum[i+ml]
		}) + ml
		// upper
		m2 = sort.Search(mr-m1, func(i int) bool {
			return 2*sum[i+m1] > sum[r]+left
		}) + m1
		ans += m2 - m1
		ans = ans % (1e9 + 7)
	}
	return ans
}
```



