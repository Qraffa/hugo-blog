---
title: "Alg 摆动排序"
date: 2020-11-29T19:33:49+08:00
tags:
- 算法
categories: 
- 算法
---

## 摆动排序

### 题目描述

给定一个无序的数组 `nums`，将它重新排列成 `nums[0] < nums[1] > nums[2] < nums[3]...` 的顺序。

**示例 1:**

```
输入: nums = [1, 5, 1, 1, 6, 4]
输出: 一个可能的答案是 [1, 4, 1, 5, 1, 6]
```

<!--more-->

### 思路

首先该数组满足类似于小，大，小，大...类似的元素关系。那么假设对于一个有序的数组，我们可以考虑按一般划分，将左边部分填充到小的位置，将右边部分填充到大的位置。

但在这题上，我们并不需要关系每个小之间的关系，也就是说只要能将小和大分为两部分就可以了，不需要再排序小或大内部的元素。

因此考虑按快排思路，以中间的基准数进行调整，就可以将数组分为一半小和一半大。

除此之外要需要考虑到中间元素较多的情况，比如[1,2,3,3,3,4]，分为两部分，[1,2,3]和[3,3,4]，直接进行填位，就变成[1,3,2,3,3,4]，不符合题目要求。因此需要将中间重复的元素尽可能分离，所以讲两个数组倒序，变成[3,2,1]和[4,3,3]，填位变成[3,4,2,3,1,3]

### code

```go
func wiggleSort(nums []int) {
    k := (len(nums) + 1) / 2
    var l, r, lt, gt int
    l, r = 0, len(nums)
    lt, gt = tripleSort(nums, l, r)
    for true {
        if len(nums)-k >= lt && len(nums)-k < gt {
            break
        }
        if gt <= len(nums)-k {
            l = gt
            lt, gt = tripleSort(nums, l, r)
        } else {
            r = lt
            lt, gt = tripleSort(nums, l, r)
        }
    }
    ans, i := make([]int, len(nums)), 0
    for 2*i < len(nums) {
        ans[2*i] = nums[(len(nums)-1)/2-i]
        if 2*i+1 < len(nums) {
            ans[2*i+1] = nums[len(nums)-1-i]
        }
        i++
    }
    for k, v := range ans {
        nums[k] = v
    }
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



