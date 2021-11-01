---
title: "最大矩形"
date: 2021-11-02T00:32:02+08:00
url: maximal-rectangle
tags:
- 算法
categories:
- 算法 
---

## 最大矩形

### 题目描述

给定一个仅包含 0 和 1 、大小为 rows x cols 的二维二进制矩阵，找出只包含 1 的最大矩形，并返回其面积。

 

示例 1：


输入：matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
输出：6
解释：最大矩形如上图所示。
示例 2：

输入：matrix = []
输出：0
示例 3：

输入：matrix = [["0"]]
输出：0
示例 4：

输入：matrix = [["1"]]
输出：1
示例 5：

输入：matrix = [["0","0"]]
输出：0

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/maximal-rectangle
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 思路

1. 对于每个单元格，维护从上往下的连续的1的个数
2. 遍历所有单元格，对于每个单元格，计算从左往右的连续的1的个数作为长，再从下往上遍历该列，以列的步数作为宽，维护长取最小值，求面积维护答案

### code

```go
func maximalRectangle(matrix [][]byte) int {
	for i := 0; i < len(matrix); i++ {
		for j := 0; j < len(matrix[i]); j++ {
			matrix[i][j] = matrix[i][j] - 48
		}
	}
  // 维护列上的连续1的个数
	for i := 0; i < len(matrix); i++ {
		for j := 1; j < len(matrix[i]); j++ {
			if matrix[i][j] != 0 && matrix[i][j-1] != 0 {
				matrix[i][j] = matrix[i][j-1] + 1
			}
		}
	}
	var ans int = 0
	for i := 0; i < len(matrix); i++ {
		for j := 0; j < len(matrix[i]); j++ {
			var ml int = max(len(matrix), len(matrix[0])) + 1
			ans = max(ans, int(matrix[i][j]))
			ml = min(ml, int(matrix[i][j]))
      // 从下往上遍历
			for k := i - 1; k >= 0 && matrix[k][j] != 0; k-- {
        // 维护最小长
				ml = min(ml, int(matrix[k][j]))
        // 计算面积
				ans = max(ans, (i-k+1)*ml)
			}
		}
	}
	return ans
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}
```

