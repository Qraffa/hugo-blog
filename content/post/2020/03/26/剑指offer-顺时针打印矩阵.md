---
title: 剑指offer-顺时针打印矩阵
date: 2020-03-26 00:00:05
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[顺时针打印矩阵](https://www.nowcoder.com/practice/9b4c81a02cd34f76be2659fa0d54342a?tpId=13&tqId=11172&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下4 X 4矩阵： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.

<!--more-->
### 思路:

​	我们首先定义4个方向,用来表示接下来往哪边走下一步,然后定义一个标记数组用来表示该位置是否走过

​	方向顺序为`右,下,左,上`

​	然后从(0,0)开始遍历,每次我们先以现在的指示方向试探前进,判断下一步能不能走,即判断下一步有没有出界,有没有被走过.

		1. 如果下一步能走,那么将该位置加进链表,标记该位走过,(x,y)坐标移动,总数+1
  		2. 如果下一步不能走,那么修改方向,再执行1的后续步骤

### code

```java
public ArrayList<Integer> printMatrix(int [][] matrix) {
    int[][] turn={
        {0,1},{1,0},{0,-1},{-1,0}
    };
    if(matrix==null){
        return null;
    }
    ArrayList<Integer> list=new ArrayList<>();
    int row=matrix.length;
    int col=matrix[0].length;
    int cnt=0,sum=0,x=0,y=0;
    int[][] vis=new int[row][col];
    for(int i=0;i<row;++i) {
        for(int j=0;j<col;++j) {
            vis[i][j]=0;
        }
    }
    while(sum<row*col) {
        int tmpx=x+turn[cnt][0];
        int tmpy=y+turn[cnt][1];
        if(tmpx<0||tmpx>=row||tmpy<0||tmpy>=col||vis[tmpx][tmpy]!=0) {
            cnt=(cnt+1)%4;
        }
        list.add(matrix[x][y]);
        vis[x][y]=1;
        x=x+turn[cnt][0];
        y=y+turn[cnt][1];
        sum++;
    }
    return list;
```



​	