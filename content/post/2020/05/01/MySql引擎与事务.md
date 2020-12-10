---
title: MySql引擎与事务
date: 2020-05-01 18:07:05
tags:
- mysql
categories:
- mysql
---

## mysql引擎

查询支持的引擎

```mysql
show engines;
```

查询某个表使用的引擎

```mysql
show table status like 'article' ;
```

<!--more-->

修改表引擎

```mysql
alter table 表名 engine = 引擎名
```

### 两种主要引擎MyISAM和InnoDB

- InnoDB支持事务
- InnoDB支持表级锁和行级锁,MyISAM只有表级锁
- InnoDB支持外键约束
- InnoDB是聚集索引,MyISAM是非聚集索引
  - .frm文件是表结构文件,每个表都有
  - InnoDB的存储文件为.ibd文件
  - MyISAM的存储文件为.MYD文件(数据文件)和.MYI文件(索引文件)
- InnoDB内部没有数据表的行数,因此`select count(*) from table`查询数据表的行数时,需要扫描全表.MyISAM内部是有行数的.
- MyISAM允许没有主键.InnoDB在没有主键的情况下,会自动生成主键.
- 在删除表时,MyISAM会重建表,InnoDB是一行一行的删除的.可以考虑使用`TRUNCATE TABLE`

## mysql事务

查询当前事务隔离级别

```mysql
SELECT @@tx_isolation;
```

修改事务隔离级别

```mysql
SET [SESSION|GLOBAL] TRANSACTION ISOLATION LEVEL [READ UNCOMMITTED|READ COMMITTED|REPEATABLE READ|SERIALIZABLE]
```

查看事务是否自动提交

```mysql
show variables like '%autocommit%';
```

#### 事务的四个特性

1. 原子性:事务是最小的执行单位,要么全部完成,要么全部不执行
2. 一致性:执行事务前后,数据保持一致
3. 隔离性:一个事务不会被其他事务干扰
4. 持久性:一个事务提交后,对数据库的修改是永久的.

#### 并发事务的四个问题

- **脏读**

  事务T1访问数据,然后事务T2对该数据进行修改,但未提交,此时T1再次访问数据,会访问到T2修改但未提交的数据.

- **丢失修改**

  前一个事务的修改可能被后一个事务的修改覆盖,那么前一个事务的修改就丢失了.

- **不可重复读**

  事务T1访问数据,然后事务T2对该数据进行修改并且提交,此时T1再次访问数据,会访问到T2修改且提交的数据

- **幻读**

  事务T1访问数据,例如`select *`,假设我们发现没有主键为5的数据,此时事务T2对该表插入了主键为5的一条数据,然后T1再次尝试插入数据却发现主键重复.但`select *`的结果仍然是没有主键为5的.

#### 四种事务隔离级别

- **读未提交(RU)**

  会导致脏读、不可重复读、幻读

- **读已提交(RC)**

  会导致不可重复读、幻读

- **可重复读(RR)**

  事务T1访问数据,然后事务T2对该数据进行了修改并且提交,此时T1再次访问数据,该数据不会是事务T2修改的数据,而仍然是事务T1第一次读取到的数据.因此称可重复读.**仍然可能存在幻读**

- **可串行化(S)**

  事务需要依次执行,事务T1对数据进行update操作,但未提交,然后事务T2尝试对该数据进行update操作,会发生阻塞等待,需要等待事务T1事务结束才能继续执行.不会产生幻读

#### Undo Log和Redo Log

>**Undo Log主要用于实现回滚和MVCC**
>
>**Redo Log主要用户实现持久化,用于数据库的崩溃恢复**

**Undo Log**

所有对数据的修改都会写入回滚日志文件中.

当事务回滚时,会通过回滚日志逻辑地修改数据库.例如事务执行insert语句,回滚时,会执行对应的delete语句.update语句也会执行相反的update语句.

**Redo Log**

包含了内存中的缓冲日志和磁盘中的日志文件.

当有数据进行修改时,会就修改的数据记录写入缓冲日志中,当事务提交时,会将缓冲日志刷会磁盘的日志文件.

**示例:**

假设有A、B两个数据，值分别为1,2.

1. 事务开始
2. 记录A=1到undo log
3. 修改A=3
4. 记录A=3到 redo log
5. 记录B=2到 undo log
6. 修改B=4
7. 记录B=4到redo log
8. 将redo log写入磁盘
9. 事务提交