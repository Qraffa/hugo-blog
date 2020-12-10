---
title: 剑指offer-和为S的连续正数序列
date: 2020-05-03 15:11:58
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[和为S的连续正数序列](https://www.nowcoder.com/practice/c451a3fd84b64cb19485dad758a55ebe?tpId=13&tqId=11194&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述

小明很喜欢数学,有一天他在做数学作业时,要求计算出9~16的和,他马上就写出了正确答案是100。但是他并不满足于此,他在想究竟有多少种连续的正数序列的和为100(至少包括两个数)。没多久,他就得到另一组连续正数和为100的序列:18,19,20,21,22。现在把问题交给你,你能不能也很快的找出所有和为S的连续正数序列? Good Luck!

<!--more-->
### 思路

**思路一:**

​	直接通过等差数列的求和公式.枚举序列的第一个元素,设第一个元素为h,最后一个元素为t,和为s

​	那么会有$(h+t)*(t-h+1)/2 = 2*s$

​	因此可求出t解,再用求和公式验证一遍,如果验证成功,则说明这段序列满足,否则继续枚举

**思路二:**

​	剑指offer上的思路.

​	维护一个队列,如果队列的和小于sum,那么队尾添加新元素.否则,删除队头元素.

​	当队列和为sum时,此时队列中的元素序列满足条件.

### code

```java
// 思路一
public static ArrayList<ArrayList<Integer> > FindContinuousSequence(int sum) {
    ArrayList<ArrayList<Integer>> lists = new ArrayList<>();
    if(sum == 1) {
        return lists;
    }
    int up = (int)Math.ceil(sum/2.0);
    for (int h = 1; h <= up; h++) {
        int delta = 1-4*h+4*h*h+8*sum;
        if(delta < 0){
            continue;
        } else {
            int t = (int)((-1+Math.sqrt(delta))/2);
            int s = (h+t)*(t-h+1)/2;
            if(s == sum) {
                ArrayList<Integer> list = new ArrayList<>();
                for (int i = h; i <= t ; i++) {
                    list.add(i);
                }
                lists.add(list);
            }
        }
    }
    return lists;
}
```

```java
// 思路二
public static ArrayList<ArrayList<Integer> > FindContinuousSequence2(int sum) {
    ArrayList<ArrayList<Integer>> lists = new ArrayList<>();
    if(sum == 1) {
        return lists;
    }
    int s = 0;
    int up = (int)Math.ceil(sum/2.0);
    Queue<Integer> queue = new LinkedList<>();
    // 这里需要up+1次,来避免最后一次无法判断
    for (int i = 1; i <= up+1;) {
        if(s < sum) {
            queue.offer(i);
            s += i++;
        } else if(s > sum) {
            s -= queue.poll();
        } else {
            ArrayList<Integer> list = new ArrayList<>();
            list.addAll(queue);
            lists.add(list);
            queue.offer(i);
            s += i++;
        }
    }
    return lists;
}
```

