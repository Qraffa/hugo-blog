---
title: "Alg 第一个未出现的正数"
date: 2020-09-10T18:03:51+08:00
tags:
- 算法
categories: 
- 算法
---

## 第一个未出现的正数

### 题目描述

给你一个未排序的整数数组，请你找出其中没有出现的最小的正整数。

<!--more-->

### 思路：

时间复杂度要求O(n)，空间复杂度要求O(1)

最开始思考方向有偏差，尝试用标记，无法实现。

正确思路：考虑使用原地排序，这里不是严格的排序，这里只需要将元素值填入对应位置即可，对于大于数组大小和非正数，直接抛弃即可。

例如：3,4,-1,1

原地排序变成：1,0,3,4

然后再遍历一次，找出其中值为0的下标，即可。

### code

```go
func firstMissingPositive(nums []int) int {
  lens := len(nums)
  for i := 0; i < lens; {
    // 大于数组长度或非正数，置为0，pass
    if nums[i] > lens || nums[i] <= 0 {
      nums[i] = 0
      i++
      continue
    }
    // 已经在对应位置，不操作，pass
    if nums[i] == i+1 {
      i++
      continue
    }
    // 尝试将该位置元素交换到对应的位置，交换完可能还需要继续对该位置进行交换，因此下标i不后移
    // nums[t] != t+1 这里是避免重复的元素值，无限交换，因此如果尝试交换的目标位置上，也已经是对应值
    // 那么就执行else，将该位置与0。
    t := nums[i] - 1
    if nums[t] >= 1 && nums[t] <= lens && nums[t] != t+1 {
      nums[t], nums[i] = nums[i], nums[t]
      continue
    } else {
      // 目标位置的值大于数组长度或非正数，或已经是对应值。
      nums[i], nums[t] = 0, nums[i]
      continue
    }
  }
  // 遍历找到值为0的下标
  for i := 0; i < lens; i++ {
    if nums[i] == 0 {
      return i + 1
    }
  }
  // for循环中未找到，表示全排序好，那结果为数组长度+1
  return lens + 1
}
```