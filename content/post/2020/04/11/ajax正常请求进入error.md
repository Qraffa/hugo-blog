---
title: ajax正常请求进入error
date: 2020-04-11 00:00:00
tags:
- ajax
categories:
- ajax
---

### ajax正常请求进入error

**问题**:ajax正常请求和返回了,却进入error方法中,而不是success

<!--more-->

**解决**:原因是ajax请求中设置了`dataType: "text"`,删除这个就好了