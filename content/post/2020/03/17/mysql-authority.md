---
title: mysql用户没有权限的问题
date: 2020-03-17 20:08:43
tags:
- mysql
categories:
- mysql
---

## 问题：之前重新安装mysql删除了超级用户root，新建了用户却忘记赋予权限，导致无法新建立数据库

<!--more-->

## 解决

- **首先停止mysql服务**

    ```cmd
    service mysql stop
    ```

- **使用跳过权限表启动mysql**  
    大多数方法提及是更改配置文件中的属性，但我的配置文件没有内容。
    因此选择别的方式启动，使用一下命令启动

    ```cmd
     mysqld_safe --skip-grant-tables
    ```

  - 启动过程遇到如下问题：
      1. 报错一：

          ```cmd
          mysqld_safe Directory ‘/var/run/mysqld’ for UNIX socket file don’t exists
          ```

          错误：提示文件不存在
          解决：创建一个文件即可
      2. 报错二：

          ```cmd
          mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
          ```

          错误：好像mysql启动时无法创建pid文件
          解决：
           1. 首先检查该文件目录`/var/run/mysqld`是否存在,不存在则创建，我的已是存在的
           2. 查看该文件目录的拥有者和属组，我的是root，因此改为mysql

              ```cmd
              sudo chown -R  mysql:mysql /var/run/mysqld/
              ```

    至此，跳过权限表的mysql启动成功

- **重新赋予用户权限**
  - 报错失败：

    ```cmd
    The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement
    ```

    解决：刷新权限表

    ```cmd
    flush privileges;
    ```

    重新赋予用户权限成功

- 至此再以正常的方式重启mysql即可

---

## 参考

[https://www.cnblogs.com/mumue/p/3816185.html](https://www.cnblogs.com/mumue/p/3816185.html)  
[https://blog.csdn.net/z_yttt/article/details/73650495](https://blog.csdn.net/z_yttt/article/details/73650495)  
[https://www.jb51.net/article/117137.htm](https://www.jb51.net/article/117137.htm)  
[https://www.cnblogs.com/qianzf/p/6995376.html](https://www.cnblogs.com/qianzf/p/6995376.html)

