---
title: 阿里云服务器mysql无法连接
date: 2020-03-17 22:27:45
tags:
- mysql
categories: 
- mysql
---

## 阿里云服务器mysql无法连接

- 问题：在阿里云服务器上部署mysql服务后，想通过datagrip连接数据库。出现无法连接的问题。
- 解决：
    1. 检查阿里云mysql启动情况,已经启动，没有问题
    2. 使用telnet测试3306端口是否正常，发现无法连接
    3. 查看3306端口的连接情况

    <!--more-->

        ```cmd
        netstat -anp | grep 3306
        ```
    
        发现3306端口只在监听本地ip`127.0.0.1`
    
        ![avatar](./mysql-netstat.png)
        **此为修改后的结果，修改前箭头处为127.0.0.1**
    4. 更改mysql的监听地址，发现my.cnf空空如也
    5. 在my.cnf添加以下内容  
        ![avatar](./mysql-mycnf.png)
    6. 重启mysql服务
    7. 完成，可以正常连接