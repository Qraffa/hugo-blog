---
title: "Alg 数组中的第k个最大元素"
date: 2020-11-29T18:47:44+08:00
tags:
- 算法
categories: 
- 算法
---

## 数组中的第k个最大元素

### 题目描述

在未排序的数组中找到第 **k** 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

<!--more-->

### 思路

类似于取数组中前k大数的问题。

方法一：维护大小为k的最小堆，对于新元素，若小于堆顶则跳过，若大于等于堆顶则替换堆顶元素，重新调整堆，最后堆顶元素就是答案

方法二：使用快排思路，对于快排一次调整可以将一个元素调整到正确位置，因此可以二分调整。这里使用三路快排。

#### 三路快排

[https://www.cnblogs.com/deng-tao/p/6536302.html](https://www.cnblogs.com/deng-tao/p/6536302.html)

三路快排思路是对于基准数k，做一次调整后将数组分为<k，=k，>k三部分。然后递归调整<k和>k的部分。

```go
// 普通快排
func quickSort(nums []int, l, r int) {
	if l >= r {
		return
	}
	q, i, j := nums[l], l, r-1
	for i < j {
		// 从后往前找小于基准数的位置
		for i < j && nums[j] >= q {
			j--
		}
		// 从前往后找大于基准数的位置
		for i < j && nums[i] <= q {
			i++
		}
		// 交换小于和大于的元素，保证左边数组为小于基准数，右边数组大于基准数
		if i < j {
			nums[i], nums[j] = nums[j], nums[i]
		}
	}
	// 将基准数调整到合适位置。即与小于数组的最后一位交换。
	nums[l], nums[i] = nums[i], nums[l]
	quickSort(nums, l, i)
	quickSort(nums, i+1, r)
}

// 三路快排
func tripleQuickSort(nums []int, l, r int) {
	if l >= r {
		return
	}
	// 数组首先被划分为以下部分
	// (l,lt]	小于基准数的部分
	// (lt,i)	等于基准数的部分
	// [i,gt)	待遍历元素
	// [gt,r)	大于基准数的部分
	// 起始状态，lt指向数组首元素，表示区间空，gr指向数组尾，表示区间空
	// i 表示当前遍历元素的下标
	q, lt, i, gt := nums[l], l, l+1, r
	for i < gt {
		// 元素小于基准数，则将i位置元素交换到小于区间，同时扩大小于区间的范围
		if nums[i] < q {
			nums[lt+1], nums[i] = nums[i], nums[lt+1]
			lt++
			i++
		// 元素大于基准数，则将i位置元素交换到大于区间，同时扩大大于区间的范围
		} else if nums[i] > q {
			nums[gt-1], nums[i] = nums[i], nums[gt-1]
			gt--
		// 对于相等的元素，直接往下一位，表示扩大等于区间的范围
		} else {
			i++
		}
	}
	// 将基准元素交换到小于空间的最后一位上
	nums[lt], nums[l] = nums[l], nums[lt]
	// 因此在一次调整之后，数组的区间应该变成：
	// [l,lt)	小于基准数的部分
	// [lt,gt)	等于基准数的部分
	// [gt,r)	大于基准数的部分
	tripleQuickSort(nums, l, lt)
	tripleQuickSort(nums, gt, r)
}
```

补充一下：

对于遇到小于基准数的位置和大于基准数的位置，i是否自增的问题。

- 对于小于基准数的情况，首先一定有lt<i的关系
  - 等于区间为空，那么此时满足lt+1=i，那么交换语句元素位置没有变化，因此扩大小于区间，即lt++，此时lt=i，i需要后移，否则死循环了
  - 等于区间不为空，那么此时满足lt+1<i，那么交换语句相当于将等于区间的第一个元素与当前i位置元素交换，因此扩大小于区间，即lt++，此时i位置的元素变成了等于基准数的元素，所以直接i++
- 对于大于基准数的情况，由于是将gt-1位置上的元素与i位置元素交换，因此并不清楚gt-1位置上的元素是什么关系，因此i不能后移，需要进入下一次判断。

### code

```go
func findKthLargest(nums []int, k int) int {
    var l, r, lt, gt int
    l, r = 0, len(nums)
    lt, gt = tripleSort(nums, l, r)
    for true {
        if len(nums)-k >= lt && len(nums)-k < gt {
            return nums[lt]
        }
        if gt <= len(nums)-k {
            l = gt
            lt, gt = tripleSort(nums, l, r)
        } else {
            r = lt
            lt, gt = tripleSort(nums, l, r)
        }
    }
    return 0
}
func tripleSort(nums []int, l, r int) (int, int) {
    if l >= r {
        return -1, -1
    }
    q, lt, i, gt := nums[l], l, l+1, r
    for i < gt {
        if nums[i] < q {
            nums[lt+1], nums[i] = nums[i], nums[lt+1]
            lt++
            i++
        } else if nums[i] > q {
            nums[gt-1], nums[i] = nums[i], nums[gt-1]
            gt--
        } else {
            i++
        }
    }
    nums[lt], nums[l] = nums[l], nums[lt]
    return lt, gt
}
```

