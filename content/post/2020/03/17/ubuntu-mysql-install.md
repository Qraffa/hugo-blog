---
title: ubuntu18.04安装mysql及密码问题
date: 2020-03-17 20:05:54
tags:
- ubuntu
- mysql
categories:
- mysql
---

# ubuntu18.04安装mysql及密码问题

## 1.安装mysql

```cmd
sudo apt-get install mysql-server
sudo apt-get insatll mysql-client
```

## 2.删除mysql及残留文件

```cmd
sudo apt-get autoremove --purge mysql-server
sudo apt-get remove mysql-server
sudo apt-get autoremove mysql-server
sudo apt-get remove mysql-common
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
```

<!--more-->

## 3.登录权限问题

```cmd
mysql -u root -p
```

出现错误

```cmd
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
```

---

```cmd
sudo mysql -u root -p
```

添加sudo启动后可以正常打开.

## 4.删除及添加新用户

查看当前用户

```sql
SELECT User,Host FROM mysql.user;
```

```cmd
+------------------+-----------+
| User             | Host      |
+------------------+-----------+
| debian-sys-maint | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
4 rows in set (0.00 sec)
```

删除root账号

```sql
DROP user 'root'@'localhost';
```

添加新用户

```sql
CREATE USER 'root'@'%' IDENTIFIED BY '123456';
```

---

出现错误

```cmd
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

原因:设置自定义密码时,过于简单,不符合密码策略

解决:

1.查看密码策略

```sql
SHOW VARIABLES LIKE 'validate_password%';
```

```cmd
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.01 sec)
```

2.设置密码验证强度等级`validate_password_policy`为LOW

```sql
set global validate_password_policy=LOW;
```

3.设置密码长度为6位`validate_password_length`

```sql
set global validate_password_length=6;
```

4.再次添加新用户可以成功

---

授权

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
```

```sql
FLUSH PRIVILEGES;
```

添加用户完成
可以正常登录

```cmd
mysql -uroot -p
```

---

flush privileges 命令的作用是将当前 user 和 privilige 表中的用户信息/权限设置从 mysql 库 (MySQL数据库的内置库) 中提取到内存里。MySQL用户数据和权限出现修改后，希望在"不重启MySQL服务"的情况下直接生效，就需要执行这个命令。
在修改某个帐号的设置后，避免重启，那么 flush privileges 之后就可以使权限设置生效。

---

关于 mysql 密码策略相关参数:

1. validate_password_length  固定密码的总长度；
2. validate_password_dictionary_file 指定密码验证的文件路径；
3. validate_password_mixed_case_count  整个密码中至少要包含大/小写字母的总个数；
4. validate_password_number_count  整个密码中至少要包含阿拉伯数字的个数；
5. validate_password_policy 指定密码的强度验证等级，默认为 MEDIUM；
6. validate_password_special_char_count 整个密码中至少要包含特殊字符的个数；
7. 关于validate_password_policy的三个强度等级:
   1. 0/LOW:只验证长度；
   2. 1/MEDIUM:验证长度、数字、大小写、特殊字符；
   3. 2/STRONG:验证长度、数字、大小写、特殊字符、字典文件；

---

## 参考

[https://blog.csdn.net/study_in/article/details/86721468](https://blog.csdn.net/study_in/article/details/86721468)
[https://blog.csdn.net/beguile/article/details/88908278](https://blog.csdn.net/beguile/article/details/88908278)
[https://blog.csdn.net/hello_world_qwp/article/details/79551789](https://blog.csdn.net/hello_world_qwp/article/details/79551789)
[https://blog.csdn.net/qingyuanluofeng/article/details/51508880](https://blog.csdn.net/qingyuanluofeng/article/details/51508880)
