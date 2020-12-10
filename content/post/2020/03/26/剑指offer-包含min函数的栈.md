---
title: 剑指offer-包含min函数的栈
date: 2020-03-26 00:00:05
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[包含min函数的栈](https://www.nowcoder.com/practice/4c776177d2c04c2494f2555c9fcc1e49?tpId=13&tqId=11173&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。

注意：保证测试中不会当栈为空的时候，对栈调用pop()或者min()或者top()方法。

<!--more-->
### 思路:

​	开两个栈,一个存原始数据,一个存当前时刻最小值,两个栈保持同步操作

### code

```java
Stack<Integer> stack=new Stack<>();
Stack<Integer> minStack=new Stack<>();

public void push(int node) {
    stack.push(node);
    if(stack.size()==1) {
        minStack.push(node);
    } else {
        minStack.push(Math.min(minStack.peek(),node));
    }
}

public void pop() {
    stack.pop();
    minStack.pop();
}

public int top() {
    return stack.peek();
}

public int min() {
    return minStack.peek();
}
```

