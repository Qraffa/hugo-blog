---
title: "Alg 数据流的中位数"
date: 2020-11-26T00:29:13+08:00
tags:
- 算法
categories: 
- 算法
---

## 数据流的中位数

### 题目描述

中位数是有序列表中间的数。如果列表长度是偶数，中位数则是中间两个数的平均值。

例如，

[2,3,4] 的中位数是 3

[2,3] 的中位数是 (2 + 3) / 2 = 2.5

设计一个支持以下两种操作的数据结构：

void addNum(int num) - 从数据流中添加一个整数到数据结构中。
double findMedian() - 返回目前所有元素的中位数。

<!--more-->

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/find-median-from-data-stream
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 思路

对于一个有序数组arr，对其较小的一半元素，构建一个最大堆，对其较大的一半元素，构建一个最小堆，因此求中位数时：

1. 如果两个堆大小一致，则中位数为(最大堆顶+最小堆顶)/2
2. 如果最大堆大，则中位数为最大堆对顶，否则为最小堆堆顶

满足以下要求来维护两个堆：

1. 最大堆中保存的是有序数组中较小的一部分，最小堆中保存的是较大的一部分，即最小堆中所有元素需要大于最大堆中所有元素
2. 两个堆大小之差需要保证在小于等于1

### code

```go
type MedianFinder struct {
	MinHeap, MaxHeap []int
}

/** initialize your data structure here. */
func Constructor() MedianFinder {
	return MedianFinder{
		MinHeap: make([]int, 0),
		MaxHeap: make([]int, 0),
	}
}

func (this *MedianFinder) AddNum(num int) {
	if len(this.MinHeap) == 0 {
		this.MinHeap = append(this.MinHeap, num)
		return
	}
	if len(this.MaxHeap) == 0 {
		this.MaxHeap = append(this.MaxHeap, num)
		return
	}
	if len(this.MinHeap) == 1 && len(this.MaxHeap) == 1 {
		if this.MinHeap[0] < this.MaxHeap[0] {
			this.MinHeap[0], this.MaxHeap[0] = this.MaxHeap[0], this.MinHeap[0]
		}
	}
	// 如果添加元素大于最小堆堆顶，则向最小堆堆顶添加。
	if num > this.MinHeap[0] {
		// 需要维护两个堆，保证两个堆的大小之差小于等于1
		// 因此最小堆大小已经大于最大堆时，需要先将最小堆堆顶元素添加到最大堆上
		if len(this.MinHeap)-len(this.MaxHeap) == 1 {
			// 将最小堆堆顶元素添加到最大堆上
			this.MaxHeap = append(this.MaxHeap, this.MinHeap[0])
			// 调整堆结构，保证最大堆的性质
			floatMaxHeap(this.MaxHeap, len(this.MaxHeap)-1)
			// 添加新元素
			this.MinHeap[0] = num
			// 调整堆结构，保证最小堆的性质
			adjustMinHeap(this.MinHeap, 0)
		} else {
			// 不需要交换堆顶元素时，直接在尾部添加新元素，调整堆结构
			this.MinHeap = append(this.MinHeap, num)
			floatMinHeap(this.MinHeap, len(this.MinHeap)-1)
		}
	} else {
		if len(this.MaxHeap)-len(this.MinHeap) == 1 {
			this.MinHeap = append(this.MinHeap, this.MaxHeap[0])
			floatMinHeap(this.MinHeap, len(this.MinHeap)-1)
			this.MaxHeap[0] = num
			adjustMaxHeap(this.MaxHeap, 0)
		} else {
			this.MaxHeap = append(this.MaxHeap, num)
			floatMaxHeap(this.MaxHeap, len(this.MaxHeap)-1)
		}
	}
	// 在添加完元素后，可能出现最大堆堆顶元素大于最小堆堆顶元素的情况，
	// 最大堆---较小一部分元素，最小堆---较大一部分元素，
	// 因此交换两个堆的堆顶元素，以保证上一条。
	if this.MaxHeap[0] > this.MinHeap[0] {
		this.MinHeap[0], this.MaxHeap[0] = this.MaxHeap[0], this.MinHeap[0]
		adjustMaxHeap(this.MaxHeap, 0)
		adjustMinHeap(this.MinHeap, 0)
	}
}

func (this *MedianFinder) FindMedian() float64 {
	if len(this.MaxHeap) == len(this.MinHeap) {
		return (float64(this.MinHeap[0]) + float64(this.MaxHeap[0])) / 2
	} else if len(this.MinHeap) > len(this.MaxHeap) {
		return float64(this.MinHeap[0])
	}
	return float64(this.MaxHeap[0])
}

// 自下往上调整堆
func floatMaxHeap(heap []int, i int) {
	tmp := heap[i]
	for k := (i - 1) / 2; k >= 0; k = (k - 1) / 2 {
		if heap[k] < tmp {
			i, heap[i] = k, heap[k]
		}
		if k-1 < 0 {
			break
		}
	}
	heap[i] = tmp
}

// 自下往上调整堆
func floatMinHeap(heap []int, i int) {
	tmp := heap[i]
	for k := (i - 1) / 2; k >= 0; k = (k - 1) / 2 {
		if heap[k] > tmp {
			i, heap[i] = k, heap[k]
		}
		if k-1 < 0 {
			break
		}
	}
	heap[i] = tmp
}

// 自上往下调整堆
func adjustMaxHeap(heap []int, i int) {
	tmp := heap[i]
	for k := i*2 + 1; k < len(heap); k = k*2 + 1 {
		if k+1 < len(heap) && heap[k+1] > heap[k] {
			k++
		}
		if heap[k] > tmp {
			i, heap[i] = k, heap[k]
		}
	}
	heap[i] = tmp
}

// 自上往下调整堆
func adjustMinHeap(heap []int, i int) {
	tmp := heap[i]
	for k := i*2 + 1; k < len(heap); k = k*2 + 1 {
		if k+1 < len(heap) && heap[k+1] < heap[k] {
			k++
		}
		if heap[k] < tmp {
			i, heap[i] = k, heap[k]
		}
	}
	heap[i] = tmp
}
```

