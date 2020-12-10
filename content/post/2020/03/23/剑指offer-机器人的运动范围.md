---
title: 剑指offer-机器人的运动范围
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof)

### 题目描述:

地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

​	直接DFS完事

​	就是WA了好几发,不够细心,题意也读半天没搞懂

​	WA第一发,没读懂题,直接写了回溯vis,导致重复走了

​	WA第二发,没注意到k判定的是每一块的数位和,我写了个总路线的求和

​	WA第三发,数位和求错了,8+9=17,求了个8+9=17=1+7=8

​	总结一下,多是细节问题,样例的2 3 1也一开始没看懂

### code

```java
int row=0,col=0,k=0;
int[][] vis;
int[][] turn = {
    {-1,0},{0,1},{1,0},{0,-1}
};
public int movingCount(int m, int n, int k) {
    vis=new int[m][n];
    row=m;
    col=n;
    this.k=k;
    for(int i=0;i<m;++i) {
        for(int j=0;j<n;++j) {
            vis[i][j]=0;
        }
    }
    return dfs(0,0);
}
int dfs(int x,int y) {
    if(x<0||x>=row||y<0||y>=col||vis[x][y]==1||(x/10+x%10+y/10+y%10>k)) {
        return 0;
    }
    int cnt=1;
    for(int i=0;i<4;++i) {
        vis[x][y]=1;
        cnt+=dfs(x+turn[i][0],y+turn[i][1]);
    }
    return cnt;
}
```

