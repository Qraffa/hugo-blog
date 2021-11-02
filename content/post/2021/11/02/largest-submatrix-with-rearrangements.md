---
title: "重新排列后的最大子矩阵"
date: 2021-11-02T10:11:32+08:00
url: largest-submatrix-with-rearrangements
tags:
- 算法
categories:
- 算法 
---

## 重新排列后的最大子矩阵

### 题目描述

给你一个二进制矩阵 matrix ，它的大小为 m x n ，你可以将 matrix 中的 列 按任意顺序重新排列。

请你返回最优方案下将 matrix 重新排列后，全是 1 的子矩阵面积。

 

示例 1：



输入：matrix = [[0,0,1],[1,1,1],[1,0,1]]
输出：4
解释：你可以按照上图方式重新排列矩阵的每一列。
最大的全 1 子矩阵是上图中加粗的部分，面积为 4 。
示例 2：



输入：matrix = [[1,0,1,0,1]]
输出：3
解释：你可以按照上图方式重新排列矩阵的每一列。
最大的全 1 子矩阵是上图中加粗的部分，面积为 3 。
示例 3：

输入：matrix = [[1,1,0],[1,0,1]]
输出：2
解释：由于你只能整列整列重新排布，所以没有比面积为 2 更大的全 1 子矩形。
示例 4：

输入：matrix = [[0,0],[0,0]]
输出：0
解释：由于矩阵中没有 1 ，没有任何全 1 的子矩阵，所以面积为 0 。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/largest-submatrix-with-rearrangements
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 思路

1. 对于每个单元格，从下往上统计连续1的个数
2. 遍历数组，对于每一行，按之前统计数量降序排序
3. 维护最小高度，遍历计算面积

### code

```go
func largestSubmatrix(matrix [][]int) int {
  // 从下往上统计连续1的个数，作为高度
	for i := 1; i < len(matrix); i++ {
		for j := 0; j < len(matrix[i]); j++ {
			if matrix[i][j] != 0 {
				matrix[i][j] = matrix[i-1][j] + 1
			}
		}
	}
	var ans int = 0
	for i := 0; i < len(matrix); i++ {
    // 按高度降序排序
		sort.Sort(sort.Reverse(sort.IntSlice(matrix[i])))
		var mh int = len(matrix) + 1
		for j := 0; j < len(matrix[i]); j++ {
      // 维护最小高
			mh = min(mh, matrix[i][j])
			ans = max(ans, mh*(j+1))
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

