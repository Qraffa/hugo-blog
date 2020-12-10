---
title: 剑指offer-用两个栈实现队列
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof)

### 题目描述:

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

<!--more-->

### 思路:

​	直接两个栈倒一倒就行了

​	栈1用于入队,栈2用于出队

### code

```java
class CQueue {
    Stack<Integer> stack1;
    Stack<Integer> stack2;
    int size;

    public CQueue() {
        stack1=new Stack<>();
        stack2=new Stack<>();
    }

    public void appendTail(int value) {
        if(stack1.size()==0 && stack2.size()!=0) {
            while(!stack2.empty()) {
                stack1.push(stack2.pop());
            }
        }
        stack1.push(value);
        size++;
    }

    public int deleteHead() {
        if(size==0) {
            return -1;
        }
        while(!stack1.empty()) {
            stack2.push(stack1.pop());
        }
        size--;
        return stack2.pop();
    }
}
```

