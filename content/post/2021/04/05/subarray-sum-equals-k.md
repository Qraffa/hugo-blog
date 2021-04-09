---
title: "和为K的子数组"
date: 2021-04-05T17:52:25+08:00
url: subarray-sum-equals-k
tags:
- 算法
categories: 
- 算法
---

## 和为K的子数组

### 题目描述

给定一个整数数组和一个整数 k，你需要找到该数组中和为 k 的连续的子数组的个数。

示例 1 :

输入:nums = [1,1,1], k = 2
输出: 2 , [1,1] 与 [1,1] 为两种不同的情况。

说明 :

数组的长度为 [1, 20,000]。
数组中元素的范围是 [-1000, 1000] ，且整数 k 的范围是 [-1e7, 1e7]。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/subarray-sum-equals-k
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 思路

#### 思路一：

前缀和，枚举

做一次前缀和，对于第i个数，枚举子数组nums(j,i)，其中j<i

复杂度O(n^2)

#### 思路二：

前缀和，hash

做一次前缀和，并且把前缀和的值保存到hash中记次数

对于目标整数k，可以表示为k=sum(j)-sum(i)，其中i<j，因此sum(i)=sum(j)-k

对于第j个数，查找sum(i)=sum(j)-k在hash中的次数，累加即为结果

例如1，2，3，4，5；目标数k=9

前缀和为1，3，6，10，15；

当j=3时（下标以0开始），sum(i)=sum(3)-k=10-9=1，因此需要找到数组中前缀和为1的次数，为1

当j=4时，sum(i)=sum(4)-k=15-9=6，因此需要找到前缀和为3的次数，为1

因此结果为2

### code

```go
func subarraySum(nums []int, k int) int {
	sum :=make([]int,len(nums)+1)
	cnt :=make(map[int]int,len(nums)+1)
	var ans int
	cnt[0]++
	for i:=0;i<len(nums);i++ {
		sum[i+1]=sum[i]+nums[i]
		ans += cnt[sum[i+1]-k]
		cnt[sum[i+1]]++
	}
	return ans
}
```

