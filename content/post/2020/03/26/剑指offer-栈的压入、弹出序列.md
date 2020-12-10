---
title: 剑指offer-栈的压入、弹出序列
date: 2020-03-26 00:00:05
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[栈的压入、弹出序列](https://www.nowcoder.com/practice/d77d11405cc7470d82554cb392585106?tpId=13&tqId=11174&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述:

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

<!--more-->
### 思路:

​	模拟

​	设置两个指针p1,p2,用来指示两个数组的移动

​	首先将两个指针置于两个数组头部,进行以下步骤

	1. 如果pushA[p1]==popA[p2],则表示这个地方是压栈然后出栈,直接都后移即可
 	2. 如果不等
      	1. 判断当前栈顶元素是否与popA[p2]是否相同,如果相同则表明,此时是出栈过程.做个出栈,然后p2后移
      	2. 如果不相同,则表明这是入栈,直接将元素入栈,p1后移

### code

```java
public boolean IsPopOrder(int [] pushA,int [] popA) {
    if(pushA==null||pushA.length==0) {
        return true;
    }
    Stack<Integer> stack=new Stack<>();
    int len=pushA.length,p1=0,p2=0;
    for (; p1 < len;) {
        if(pushA[p1]==popA[p2]) {
            p1++;
            p2++;
        } else {
            if(!stack.empty() && stack.peek()==popA[p2]) {
                stack.pop();
                p2++;
            } else {
                stack.push(pushA[p1++]);
            }
        }
    }
    boolean flag=true;
    while(!stack.empty()) {
        if(stack.peek()!=popA[p2++]) {
            flag=false;
            break;
        }
        stack.pop();
    }
    return flag;
```



