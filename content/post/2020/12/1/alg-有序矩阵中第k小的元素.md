---
title: "Alg 有序矩阵中第k小的元素"
date: 2020-12-01T20:40:46+08:00
tags:
- 算法
categories: 
- 算法
---

## 有序矩阵中第k小的元素

### 题目描述

给定一个 *`n x n`* 矩阵，其中每行和每列元素均按升序排序，找到矩阵中第 `k` 小的元素。
请注意，它是排序后的第 `k` 小元素，而不是第 `k` 个不同的元素。

示例：

matrix = [
   [ 1,  5,  9],
   [10, 11, 13],
   [12, 13, 15]
],
k = 8,

返回 13。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/kth-smallest-element-in-a-sorted-matrix
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路

#### 思路一

按大小遍历矩阵，遍历到第k个就是结果。bfs，维护最小堆保存下一步的最小值。

```go
type Pos struct {
    x, y int
    val  int
}
func kthSmallest(matrix [][]int, k int) int {
    minHeap := make([]Pos, 0)
    vis := make([][]bool, len(matrix))
    for i := 0; i < len(matrix); i++ {
        vis[i] = make([]bool, len(matrix))
    }
    cnt := 0
    x, y := 0, 0
    for cnt < k {
        if len(minHeap) > 0 {
            minx := minHeap[0]
            cnt++
            if cnt >= k {
                return minx.val
            }
            if (minx.x+1 < len(matrix) && !vis[minx.x+1][minx.y]) && (minx.y+1 < len(matrix) && !vis[minx.x][minx.y+1]) {
                minHeap[0] = Pos{minx.x + 1, minx.y, matrix[minx.x+1][minx.y]}
                vis[minx.x+1][minx.y] = true
                adjustHeapSink(minHeap, 0)
                minHeap = append(minHeap, Pos{minx.x, minx.y + 1, matrix[minx.x][minx.y+1]})
                vis[minx.x][minx.y+1] = true
                adjustHeapFloat(minHeap, len(minHeap)-1)
            } else if minx.x+1 < len(matrix) && !vis[minx.x+1][minx.y] {
                minHeap[0] = Pos{minx.x + 1, minx.y, matrix[minx.x+1][minx.y]}
                vis[minx.x+1][minx.y] = true
                adjustHeapSink(minHeap, 0)
            } else if minx.y+1 < len(matrix) && !vis[minx.x][minx.y+1] {
                minHeap[0] = Pos{minx.x, minx.y + 1, matrix[minx.x][minx.y+1]}
                vis[minx.x][minx.y+1] = true
                adjustHeapSink(minHeap, 0)
            } else {
                minHeap[0], minHeap[len(minHeap)-1] = minHeap[len(minHeap)-1], minHeap[0]
                minHeap = minHeap[:len(minHeap)-1]
                adjustHeapSink(minHeap, 0)
            }
        } else {
            minHeap = append(minHeap, Pos{x, y, matrix[x][y]})
            vis[x][y] = true
        }
    }
    return minHeap[0].val
}
func adjustHeapSink(nums []Pos, k int) {
    tmp := nums[k]
    for i := k*2 + 1; i < len(nums); i = i*2 + 1 {
        if i+1 < len(nums) && nums[i+1].val < nums[i].val {
            i++
        }
        if nums[i].val < tmp.val {
            k, nums[k] = i, nums[i]
        }
    }
    nums[k] = tmp
}
func adjustHeapFloat(nums []Pos, k int) {
    tmp := nums[k]
    for i := (k - 1) / 2; i >= 0; i = (i - 1) / 2 {
        if nums[i].val > tmp.val {
            k, nums[k] = i, nums[i]
        }
        if i-1 < 0 {
            break
        }
    }
    nums[k] = tmp
}
```

#### 思路二

二分枚举答案，下界为矩阵最小值(matrix(0,0))，上界为矩阵最大值(matrix(n-1,n-1))。对于每一个可能的答案v，**在矩阵中统计小于等于v的数**。

**关于二分枚举的答案是否在矩阵中的问题：**

首先将问题转换一下，对于示例的数组，我们构建如下数组nums，nums[i]表示矩阵中小于等于i的个数

因此nums为[0,1,1,1,1,2,2,2,2,3,4,5,6,8,8,9]

对于查找第k小元素，问题变成在nums数组中查找第一个大于等于k的元素的位置。

- 情况一：k在nums存在，这表示二分查找总能找到第一个等于k的元素的位置。并且我们统计的是**在矩阵中统计小于等于v的数**，因此一定存在一个数属于等于的情况。例如对于上面的nums，我们统计nums(12)=6,nums(13)=8,nums(14)=8，因此我们可以看出矩阵中等于13的数有2个，等于14的数有0个，因此在二分查找nums时，找第一个大于等于k=8的位置，找到的是13，在矩阵中是存在的。
- 情况二：k在nums中不存在，这表示二分查找的结果位置是第一个大于k的位置。同理情况一，假设此时查找的k=7，因此二分查找找到的位置是第一个大于7的位置，即nums(13)=8。

**我们统计的nums是在矩阵中统计小于等于v的数**。

简单来说，对于nums数组的统计结果，nums(i)-nums(i-1)等于数i在矩阵中出现的次数。由于二分查找找的是第一个大于等于k的数，因此只会找到恰好出现一次的数，比如6，或者多个相同数的第一个，比如8。因此连续相等的数后几位是不可能被查找到的，也就是出现次数为0的数不可能被查找到。

```go
func kthSmallest(matrix [][]int, k int) int {
    l, r := matrix[0][0], matrix[len(matrix)-1][len(matrix)-1]
    var m int
    // 二分枚举答案，查找≥k的第一个元素。
    for l < r {
        m = (r-l)/2 + l
        if cnt(matrix, m) < k {
            l = m + 1
        } else {
            r = m
        }
    }
    return l
}
// 统计矩阵中≤val的个数
func cnt(matrix [][]int, val int) int {
    res, i, j := 0, 0, len(matrix)-1
    for j >= 0 {
        if matrix[i][j] <= val {
            i++
        }
        if i == len(matrix) || matrix[i][j] > val {
            res += i
            if i == len(matrix) {
                i--
            }
            j--
        }
    }
    return res
}

```



