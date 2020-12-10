---
title: git
date: 2020-03-17 16:16:59
index_img: /img/example.jpg
tags:
- git
categories: 
- git
---

## git常用命令

### clone

通过url下载到本地

```git
git clone 'url'
```

### add

添加文件到本地的暂存区

```git
git add .
```

<!--more-->

### commit

将暂存区的修改提交到本地仓库

```git
git commit -m "msg"
```

### push

将本地更新推送到远程仓库

```git
git push <远程主机名> <本地分支>:<远程分支>
```

---

```git
git push origin master
```

省略了<远程分支>, 表示将本地分支推送给与之存在追踪关系的远程分支

### pull

拉取远程仓库的代码与本地合并

```git
git pull origin master
```

如上表示拉取远程仓库的master分支并合并

---

## git用法

- 将本地修改提交到远程仓库

```git
git add .
git commit -m 'msg'
git push origin master
```

---

## git使用遇到的问题

- 当本地有文件修改无法pull时,强制覆盖本地的修改文件

```git
git fetch --all
git reset --hard origin/master
git pull origin master
```

`fetch`表示从远程仓库下载最新,但不进行合并操作
`reset`将主分支重置为刚刚获取到的最新的内容
`--hard`更改工作树中文件,以匹配`origin/master`

