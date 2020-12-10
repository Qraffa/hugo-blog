---
title: 剑指offer-旋转数组的最小数字
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

### 题目描述:

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

### 解法1:暴力

​	好像行吧,毕竟标签easy

### 解法2:

​	这题还是挺有意思的.暴力做法是O(N),用二分做O(lgN),不过二分有几个注意点

​	首先注意到数组左右两侧为有序,考虑二分

​	考虑一下几种情况来缩小范围:

​	假设i为下界,j为上界,m=(i+j)/2

  1. numbers[m]>numbers[j]

     eg: 3 4 5 6 1

     这表明m落在了左侧数组,旋转点会在它左侧,因此

     i=m+1

  2. numbers[m]<numbers[j]

     eg: 3 4 1 2 3

     这表明m落在了右侧数组,旋转点在它的位置或者它的左侧,因此

     j=m

		3. numbers[m]==numbers[j]

     eg: 1 0 1 1 1

     比较麻烦的情况

     因为这种情况下无法确定落在左侧还是右侧,

     左侧情况: 1 1 1 0 1

     右侧情况: 1 0 1 1 1

     此时,注意到,如果i到j的这段数组满足**number[i]>numbers[j]**,那么说明旋转点x会在区间(i,j]

     因此我们将下界i++,缩小范围

总结一下:结合了leetcode大神的题解做了改动的,leetcode给出的是将上界j--,当然他也指出在**[1,1,1,2,3,1]**这组特殊样例中会丢掉真正的旋转点,但该做法的返回值依然是正确的.为此,我选择从下界缩小,这样应该不会丢掉真正的旋转点

注意几个特殊样例:

**1 0 1 1 1**

**1 1 1 0 1**

**1 1 1 2 3 1**

### code

```java
public int minArray(int[] numbers) {
    int i=0,j=numbers.length-1;
    int m=0;
    while(i<=j) {
        // 注意该条件不能丢
        if(numbers[i]<numbers[j])
            return numbers[i];
        m=(i+j)/2;
        if(numbers[m]>numbers[j]) {
            i=m+1;
        } else if(numbers[m]<numbers[j]) {
            j=m;
        } else {
            i++;
        }
    }
    return numbers[m];
}
```



