---
title: 剑指offer-第一个只出现一次的字符
date: 2020-05-03 15:11:58
tags:
- 剑指offer
categories:
- 算法
---

## 剑指offer-[第一个只出现一次的字符](https://www.nowcoder.com/practice/1c82e8cf713b4bbeb2a5b31cf5b0417c?tpId=13&tqId=11187&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述

在一个字符串(0<=字符串长度<=10000，全部由字母组成)中找到第一个只出现一次的字符,并返回它的位置, 如果没有则返回 -1（需要区分大小写）.（从0开始计数）

<!--more-->

### 思路

先对字符串中的字符存入hashmap中,字符作为key,出现次数作为值

然后在遍历一次字符串,然后取出出现次数,如果为1,就return.

### code

```java
public static int FirstNotRepeatingChar(String str) {
    HashMap<Character,Integer> map = new HashMap<>(60);
    int len = str.length();
    for (int i = 0; i < len; i++) {
        char c = str.charAt(i);
        if(map.containsKey(c)) {
            map.replace(c,map.get(c)+1);
        } else {
            map.put(c,1);
        }
    }
    for (int i = 0; i < len; i++) {
        if(map.get(str.charAt(i)) == 1) {
            return i;
        }
    }
    return -1;
}
```

