---
title: 剑指offer-队列最大值
date: 2020-04-20 00:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-队列最大值

### 题目描述:

给定一个数组和滑动窗口的大小，找出所有滑动窗口里数值的最大值。例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为{4,4,6,6,6,5}； 针对数组{2,3,4,2,6,2,5,1}的滑动窗口有以下6个： {[2,3,4],2,6,2,5,1}， {2,[3,4,2],6,2,5,1}， {2,3,[4,2,6],2,5,1}， {2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}， {2,3,4,2,6,[2,5,1]}。

<!--more-->

### 思路:

维护双端队列存数组最值下标

当有元素添加,将队列中小于该元素的下标删除,然后添加进该下标

当移动指针大于首元素的小标,将队列中首元素移除

### code

```java
public static ArrayList<Integer> maxInWindows(int [] num, int size)
{
    ArrayList<Integer> list = new ArrayList<>();
    int len=num.length;
    Deque<Integer> deque = new ArrayDeque<>();
    for (int i = 0; i < len; i++) {
        if (deque.size() != 0) {
            while (deque.size() > 0 && num[deque.getLast()] < num[i]) deque.removeLast();
        }
        deque.addLast(i);
        if(deque.size()>0 && i-size+1>deque.getFirst()) deque.removeFirst();
        // 当移动指针位置大于窗口大小,可以添加进数组中.
        if(size>0&&i>=size-1) {
            list.add(num[deque.getFirst()]);
        }
    }
    return list;
}
```



