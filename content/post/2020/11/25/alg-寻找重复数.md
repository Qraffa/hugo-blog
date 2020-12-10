---
title: "Alg 寻找重复数"
date: 2020-11-25T19:18:24+08:00
tags:
- 算法
categories: 
- 算法
---

## 寻找重复数

### 题目描述

给定一个包含 n + 1 个整数的数组 nums，其数字都在 1 到 n 之间（包括 1 和 n），可知至少存在一个重复的整数。假设只有一个重复的整数，找出这个重复的数。

<!--more-->

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/find-the-duplicate-number
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

1. **不能**更改原数组（假设数组是只读的）。
2. 只能使用额外的 *O*(1) 的空间。
3. 时间复杂度小于 *O*(*n*2) 。
4. 数组中只有一个重复的数字，但它可能不止重复出现一次。

### 思路

#### 思路一：

快慢指针。以数组下标k为点，以k->nums[k]为边，做有向图，因为至少存在一个重复的整数，因此该有向图中一定存在环，且该重复数是环的入口。

问题成为寻找有向图中的环的入口节点。[[剑指offer-链表中环的入口结点]](http://qraffa.cn/2020/03/24/%E5%89%91%E6%8C%87offer-%E9%93%BE%E8%A1%A8%E4%B8%AD%E7%8E%AF%E7%9A%84%E5%85%A5%E5%8F%A3%E7%BB%93%E7%82%B9/)

#### code

```go
func findDuplicate(nums []int) int {
	f, s := 0, 0
	for true {
		f = nums[nums[f]]
		s = nums[s]
		if f == s {
			break
		}
	}
	s = 0
	for true {
		f = nums[f]
		s = nums[s]
		if f == s {
			break
		}
	}
	return f
}
```

#### 思路二：

二分查找。对于1-n中的数k，在数组中统计≤k的数的个数

- 如果统计cnt<=k，那么表示重复的数大于k
- 如果统计cnt>k，那么表示重复的数小于等于k

假设长度为n+1的数组中，只有一个重复的数，如果cnt>k，则表明k或k之前的数字出现重复了；如果cnt=k，则表明1-k恰好出现一次，因此重复数字出现在k之后；如果cnt<k，则表明1-k中有数字出现空缺，因此重复数字出现在k之后。

#### code

```go
func findDuplicate(nums []int) int {
	l, r := 1, len(nums)
	for l < r {
		m := (r-l)/2 + l
		cnt := 0
		for i := 0; i < len(nums); i++ {
			if nums[i] <= m {
				cnt++
			}
		}
		if cnt <= m {
			l = m + 1
		} else {
			r = m
		}
	}
	return r
}

```

