---
title: "Alg 滑动窗口最大值"
date: 2020-08-27T10:56:02+08:00
tags:
- 算法
categories: 
- 算法
---

## 滑动窗口最大值

### 题目描述：

给定一个数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。滑动窗口每次只向右移动一位。

返回滑动窗口中的最大值。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/sliding-window-maximum
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路

优先队列保存入队元素的下标。

1. 如果移动后，窗口范围超出队列最大元素，即队首，则移出队首。
2. 对于一个需要入队的元素，先将队列中所有小于该值的元素移出。
3. 元素入队。

### code

```go
func maxSlidingWindow(nums []int, k int) []int {
	queue := make([]int, 0)
	res := make([]int ,0)
	for i, v := range nums {
		if len(queue) > 0 {
			// 移动后，最大值位置超出窗口范围，移除。
			if i-k+1 > queue[0] {
				queue = queue[1:]
			}
			// 如果不为空，将小于入队元素的移除。
			for j, jv := range queue {
				if nums[jv] < v {
					queue = queue[:j]
					break
				}
			}
		}
		// 元素入队
		queue = append(queue, i)
		// 偏移大于窗口，结果元素入队。
		if i+1 >= k {
			res = append(res, nums[queue[0]])
		}
	}
	return res
}
```

坑：

```go
// 如果不为空，将小于入队元素的移除。
for j, jv := range queue {
  if nums[jv] < v {
    queue = queue[:j]
    break
  }
}
```

这里的遍历队列移出小于值的元素时，当数组降序排序，且窗口大小等于数组大小时，最坏情况。O(n^2)