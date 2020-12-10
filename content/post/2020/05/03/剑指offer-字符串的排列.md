---
title: 剑指offer-字符串的排列
date: 2020-05-03 15:11:58
tags:
- 剑指offer
categories:
- 算法
---
## 剑指offer-[字符串的排列](https://www.nowcoder.com/practice/fe6b651b66ae47d7acce78ffdd9a96c7?tpId=13&tqId=11180&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述

输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。

<!--more-->
### 思路

对字符串做全排列，然后将所有排列存入set中去重，在转换为arraylist

### code

```java
public ArrayList<String> Permutation(String str) {
    TreeSet<String> set = new TreeSet<>();
    ArrayList<String> list = func(str);
    set.addAll(list);
    list.clear();
    list.addAll(set);
    return list;
}

// 全排列
public ArrayList<String> func(String str) {
    ArrayList<String> list = new ArrayList<>();
    if(str != null && str.length() == 1) {
        list.add(str);
    } else{
        int len = str.length();
        String s = str;
        for (int i = 0; i < len; i++) {
            char head = s.charAt(0);
            ArrayList<String> list1 = func(s.substring(1));
            int len1 = list1.size();
            for (int i1 = 0; i1 < len1; i1++) {
                list.add(head+list1.get(i1));
            }
            // swap
            if(i != len-1) {
                StringBuilder sb = new StringBuilder(str);
                StringBuilder sb2 = new StringBuilder();
                sb2.append(sb.charAt(i+1));
                sb2.append(sb.deleteCharAt(i+1));
                s = str = sb2.toString();
            }
        }
    }
    return list;
}
```

