---
title: 剑指offer-二维数组中的查找
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof)

### 题目描述:

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

### 解法1: 暴力

### 解法2:

​	从数组的左下角或右上角开始查找,

​	以右上角为例:

​	如果target大于该位,则说明target在下面,列数++

​	如果target小于该位,则说明target在左边,行数--

### 注意:

	1. 注意数组越界问题
 	2. 输入数组可能是空的,emmmm

### code

```java
public static boolean findNumberIn2DArray(int[][] matrix, int target) {
    int row=matrix.length;
    if (row==0) {
        return false;
    }
    int col=matrix[0].length;
    int i=0,j=col-1;
    boolean flag=false;
    while(i>=0&&j>=0&&i<row&&j<col) {
        int pos=matrix[i][j];
        if(pos==target) {
            flag=true;
            break;
        } else if(pos>target) {
            j--;
        } else if(pos<target) {
            i++;
        }
    }
    return flag;
}
```

