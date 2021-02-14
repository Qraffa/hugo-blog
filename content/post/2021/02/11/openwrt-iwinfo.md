---
title: "Openwrt Iwinfo"
date: 2021-02-11T16:22:01+08:00
tags:
- openwrt
categories: 
- openwrt
---

## openwrt自编译iwinfo

### 获取iwinfo

[https://github.com/MinimSecure/iwinfo-lite](https://github.com/MinimSecure/iwinfo-lite)

这里获取的是简化版的，去掉了ubus和uci的

<!--more-->

### 编译过程

1. 先执行一次自带的makefile的编译`make`

2. 使用以下编译自己写的c文件

   ```bash
   cc  -std=gnu99 -fstrict-aliasing -Iinclude -fPIC -c -o a.o a.c
   cc -o a a.o -L. -liwinfo
   ```

3. 然后执行

### Bug

- 在ubuntu本机上，正常运行，在wrt上运行结果都是unknown。但运行make编译出来的iwinfo是正常的结果

### Debug

修改编译过程：

```bash
make
cc  -std=gnu99 -fstrict-aliasing -Iinclude -fPIC -c -o a.o a.c
cc -o a a.o -L. -liwinfo
```

### Q

#### gdb如何调试带参数的程序

1. 先使用gdb进入调试模式
2. 使用命令`set args`设置参数，可以设置多个

```bash
gdb a.out
> set args wlp3s0
```

#### gdb无法设置断点

问题：尝试在a.c文件的33行处设置断点，提示如下：

`没有符号表被读取。请使用 "file" 命令。`

其原因是生成的二进制可执行文件没有使用-g选项。

解决：在编译时加上参数`-g`

```bash
cc -g -Iinclude a.c iwinfo_utils.o iwinfo_wext.o iwinfo_wext_scan.o iwinfo_lib.o
```

#### 提示cc command not found

1. 安装gcc
2. 做个cc连接到gcc

```bash
cd /usr/bin
ln -s gcc cc
```



---

make编译结果

```bash
rm -f *.o libiwinfo.so  iwinfo
cc  -std=gnu99 -fstrict-aliasing -Iinclude -fPIC -c -o iwinfo_utils.o iwinfo_utils.c
cc  -std=gnu99 -fstrict-aliasing -Iinclude -fPIC -c -o iwinfo_wext.o iwinfo_wext.c
cc  -std=gnu99 -fstrict-aliasing -Iinclude -fPIC -c -o iwinfo_wext_scan.o iwinfo_wext_scan.c
cc  -std=gnu99 -fstrict-aliasing -Iinclude -fPIC -c -o iwinfo_lib.o iwinfo_lib.c
cc  -std=gnu99 -fstrict-aliasing -Iinclude -fPIC -c -o iwinfo_cli.o iwinfo_cli.c
cc -o libiwinfo.so iwinfo_utils.o iwinfo_wext.o iwinfo_wext_scan.o iwinfo_lib.o  -shared
cc -o iwinfo iwinfo_cli.o  -L. -liwinfo
```



---



```bash
cc -g -Iinclude a.c iwinfo_utils.o iwinfo_wext.o iwinfo_wext_scan.o iwinfo_lib.o iwinfo_nl80211.o
```



```bash
cc -g -Iinclude a.c iwinfo_utils.o iwinfo_wext.o iwinfo_wext_scan.o iwinfo_lib.o iwinfo_nl80211.o -lnl-3 -lnl-genl-3
```



```bash
cc  -std=gnu99 -fstrict-aliasing -Iinclude -fPIC -c -o a.o a.c

cc -o a a.o -L. -liwinfo
```

