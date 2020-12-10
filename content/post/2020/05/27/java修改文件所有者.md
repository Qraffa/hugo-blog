---
title: java修改文件所有者
date: 2020-05-27 16:39:53
tags:
- java
categories:
- java
---

## java修改文件所有者

最近在项目中需要对unix文件进行操作,因为通过root创建的文件其他用户是没权限访问的,因此需要修改unix的文件的所有者的组.

<!--more-->

**方法:**

使用nio包中的相关方法

```java
String strPath = String.format("%s/%s", "/home/qraffa/purple/judge", "q1w2e3d5");
// 文件路径获取path
Path path = Paths.get(strPath);
// 查找服务可用于查找用户或组名称
UserPrincipalLookupService service = FileSystems.getDefault()
    .getUserPrincipalLookupService();
// 根据名称来查找用户
UserPrincipal userPrincipal = service.lookupPrincipalByName("qraffa");
// 根据名称来查找组
GroupPrincipal groupPrincipal = service.lookupPrincipalByGroupName("qraffa");
// 获取文件的属性
PosixFileAttributeView fileAttributeView =
    Files.getFileAttributeView(path, PosixFileAttributeView.class);
// 设置文件的所有者和组
fileAttributeView.setOwner(userPrincipal);
fileAttributeView.setGroup(groupPrincipal);
// 查看文件所有者
System.out.println(fileAttributeView.getOwner());
```