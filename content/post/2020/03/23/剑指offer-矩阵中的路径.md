---
title: 剑指offer-矩阵中的路径
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof)

### 题目描述:

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

[["a","b","c","e"],
["s","f","c","s"],
["a","d","e","e"]]

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

​	直接DFS完事

​	WA一次在未标记,

​	TLE一次在找到合适路径后没有直接renturn

### code

```java
boolean flag=false;
int row=0,col=0,len=0;
char[][] board;
int[][] isGo;
String word;
int[][] turn = {
    {-1,0},{0,1},{1,0},{0,-1}
};

public boolean exist(char[][] board, String word) {
    this.board=board;
    this.word=word;
    this.row=board.length;
    this.col=board[0].length;
    this.len=word.length();
    this.isGo=new int[row][col];
    for(int i=0;i<row;++i) {
        for(int j=0;j<col;++j) {
            isGo[i][j]=0;
        }
    }
    for(int i=0;i<row;++i) {
        for(int j=0;j<col;++j) {
            if(!flag&&board[i][j]==word.charAt(0)) {
                dfs(0,i,j);
            }
        }
    }
    return flag;
}

void dfs(int step,int x,int y) {
    if(flag) {
        return;
    }
    if(step>=len) {
        flag=true;
        return;
    }
    if(x<0||x>=row||y<0||y>=col||isGo[x][y]==1||board[x][y]!=word.charAt(step)) {
        return;
    }
    for(int i=0;i<4;++i) {
        isGo[x][y]=1;
        dfs(step+1,x+turn[i][0],y+turn[i][1]);
        isGo[x][y]=0;
    }
}
```

