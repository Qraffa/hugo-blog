---
title: 剑指offer-数组中出现次数超过一半的数字
date: 2020-03-27 14:49:07
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[数组中出现次数超过一半的数字](https://www.nowcoder.com/practice/e8a1b01a2df14cb2b228b30ee6a92163?tpId=13&tqId=11181&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

<!--more-->
### 思路:

​	剑指offer书上提供的思路:

​	如果一个数出现的次数大于数组长度的一半,那么这个数的次数大于其他数出现次数之和

​	因此,设置num当前数,cnt表示当前数出现次数,若下一个数与num相同,则cnt++,否则cnt--,当cnt==0时,num等于新数,并且cnt设置为1

​	最后再检查一次,求出的num是否出现的次数是否大于数组长度的一半



​	还有个思路是类似快排的分治,

​	如果一个数出现的次数大于数组长度的一半,那么按照排序后来看,它会在中位数上.

​	选择一个基准数,按照快排思路,若他的位置是n/2,那它是中位数,若位置大于n/2,说明中位数在它左,反之在右

### code

```java
public int MoreThanHalfNum_Solution(int [] array) {
    if(array==null) {
        return 0;
    }
    int num=array[0],cnt=1;
    int len=array.length;
    for (int i = 1; i < len; i++) {
        if(array[i]!=num) {
            cnt--;
            if(cnt==0) {
                num=array[i];
                cnt=1;
            }
        } else {
            cnt++;
        }
    }
    int times=0;
    for (int i = 0; i < len; i++) {
        if(array[i]==num) {
            times++;
        }
    }
    return times>len/2?num:0;
```

