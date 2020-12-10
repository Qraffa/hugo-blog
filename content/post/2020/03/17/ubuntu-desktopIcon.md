---
title: Ubuntu18 添加桌面图标
date: 2020-03-17 20:03:32
tags: 
- ubuntu
categories:
- ubuntu
---

- ubuntu18 桌面文件存放位置  
    **/usr/share/applications**
- 桌面文件格式  
    **xxx.desktop**

<!--more-->

## 以添加一个datagrip的桌面图标为例子

1. 在 **/usr/share/applications** 下新建 **datagrip.desktop**
2. vim打开编辑

    ```bash
    # 文件头
    [Desktop Entry]

    # 编码方式
    Encoding=UTF-8

    #  程序名称
    Name=Datagrip

    # 程序执行路径
    Exec=/home/qraffa/app/DataGrip-2019.3.2/bin/datagrip.sh

    # 程序图标
    Icon=/home/qraffa/app/DataGrip-2019.3.2/bin/datagrip.svg

    # 程序类型
    Type=Application
    ```

3. 保存退出即可
