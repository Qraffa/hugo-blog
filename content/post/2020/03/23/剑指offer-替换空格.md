---
title: 剑指offer-替换空格
date: 2020-03-23 23:00:00
tags:
- 剑指offer
categories: 
- 算法
---

## 剑指offer-[替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof)

### 题目描述

请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

<!--more-->

### 思路:

​	没啥好说的

### code

```java
public String replaceSpace(String s) {
    int len=s.length();
    StringBuffer buffer = new StringBuffer();
    for(int i=0;i<len;++i) {
        if(s.charAt(i)==' ') {
            buffer.append("%20");
        } else {
            buffer.append(s.charAt(i));
        }
    }
    return buffer.toString();
}
```

