---
title: "InnoDB undo日志实验"
date: 2022-07-22T10:11:32+08:00
url: post/2022/07/22/innodb-undo
tags:
- mysql
- innodb
categories:
- mysql
---

## MySQL undo日志实验

背景：在学习undo日志时，看到一个观点：

对于update操作的undo日志，不更新主键的情况下：

- 如果被更新的列占用的存储空间未发生变化，则就地更新
- 如果被更新的列占用的存储空间发生改变，变大或变小，则先删除后插入，并且该删除时直接删除，而不是标记删除

因此本文浅浅实验下，在上述两种情况下的表空间变化，以及undo日志，并看下回滚是如何操作的。

### 环境

MySQL版本：

```mysql
mysql> select version();
+-----------+
| version() |
+-----------+
| 8.0.29    |
+-----------+
1 row in set (0.00 sec)
```

### 实验步骤概述

1. 创建表使用varchar字段
2. 添加记录，查看表空间该行数据
3. 更新该记录，但是长度相同
   1. 查看表空间该行数据
   2. 查看undo日志内容
   3. 回滚更新操作，再次查看表空间
4. 删除表重建
5. 添加记录，查看表空间该行数据
6. 更新该记录，但是长度变大
   1. 查看表空间该行数据
   2. 查看undo日志内容
   3. 回滚更新操作，再次查看表空间
7. 删除表重建
8. 添加记录，查看表空间该行数据
9. 更新该记录，但是长度变小
   1. 查看表空间该行数据
   2. 查看undo日志内容
   3. 回滚更新操作，再次查看表空间

#### 数据准备

```mysql
create table test_undo(
    id int,
    name varchar(10),
    primary key (id)
)engine=innodb charset=utf8 row_format=dynamic;
```

使用utf8编码，使用默认的dynamic行格式。

```mysql
insert into test_undo values (1,'aaa'),(2,'xxx')
```

插入两条数据，这里为了区分先删除再插入和原地更新，需要插入两条数据，然后我们操作第一条。

```shell
mysql> select TABLE_ID from information_schema.INNODB_TABLES where name like '%test_undo%';
+----------+
| TABLE_ID |
+----------+
|     1272 |
+----------+
1 row in set (0.00 sec)
```

先查出我们创建的表的table_id，这个后续查看undo日志需要用到。

### 一、测试就地更新

#### 1. 查看ibd

首先查看目前ibd文件数据页内容

```shell
hexdump -C test_undo.ibd
```

其他数据暂时不用太关心，可以看到第二个红框中显然就是第一条数据的内容'aaa'，除此之外我们还要注意到该行数据目前的trx_id是0x54d3，即21715

![image-20220720153530669](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720153530669.png)

#### 2. 修改数据

```mysql
set autocommit=0;
begin;
update test_undo set name='bbb' where id=1;
```

开启事务修改内容，这里修改的大小是和原数据一样的。但先不要提交和回滚，先卡在这里。

然后我们看一下，我们目前这个事务的trx_id，为21720，即0x54d8

```mysql
mysql> select trx_id from information_schema.INNODB_TRX;
+--------+
| trx_id |
+--------+
|  21720 |
+--------+
1 row in set (0.00 sec)
```

#### 3. 查看ibd以及undo日志

接下来查看ibd文件数据页内容，命令不再重复了。我们看到该行数据变更为了'bbb'，并且trx_id为0x54d8，也就是我们目前正在执行的事务的id。

![image-20220720154243203](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720154243203.png)

接下来查看undo日志，一般是mysql数据目录下的undo_001或undo_002。这里有几个信息我们要留意下，首先注意到该undo日志记录的table_id为1272，就是我们最开始查的该表的table_id。接下来是记录的data_trx_id为21715，即0x54d3，以及数据内容为'aaa'，也就是我们最开始插入时的内容。

![image-20220720154539174](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720154539174.png)

因此，我们可以看出对于未改变数据列大小的更新，采用原地更新的方式，将更新的数据写入到ibd数据页中，并且产生undo日志记录，记录的是修改前的内容。

#### 4. 回滚

最后我们将该事务回滚掉，rollback

然后再查看下ibd文件数据页内容，可以看到数据内容回滚到了事务前的内容。

![image-20220720155058710](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720155058710.png)

### 二、测试删除再插入

#### 1. 查看ibd

首先查看ibd文件数据页内容，也就是上面第四步的内容，这里不再重复。

除此之外，我们还需要额外关注两个字段，方便起见，这里使用工具来查看ibd文件。可以看到目前该数据页的page_free字段为0，以及page_garbage字段为0，表示目前该页没有被删除的记录的空闲空间。

![image-20220720155746013](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720155746013.png)

#### 2. 修改数据

```mysql
set autocommit=0;
begin;
update test_undo set name='ccccc' where id=1;
```

开启事务修改内容，注意这里修改的大小是比原数据大的。同样先不要提交和回滚，先卡在这里。

同样我们看一下，我们目前这个事务的trx_id，为21722，即0x54da

```shell
mysql> select trx_id from information_schema.INNODB_TRX;
+--------+
| trx_id |
+--------+
|  21722 |
+--------+
1 row in set (0.00 sec)
```

#### 3. 查看ibd以及undo日志

我们可以看到修改前的那行数据还在原处。并且在末尾我们注意到有我们update语句的新内容'ccccc'，并且trx_id为0x54da，即21722，也就是当前正在执行的事务的id

![image-20220720160133396](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720160133396.png)

除此之外，我们还要查看上面说的两个字段，page_free和page_garbage，可以看到这两个字段已经不为0了，说明第一条数据已经被认为删除掉了

![image-20220720160440054](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720160440054.png)

接下来查看undo日志，首先注意到该undo日志记录的table_id为1272，就是我们最开始查的该表的table_id。接下来是记录的data_trx_id为21715，即0x54d3，以及数据内容为'aaa'，也就是我们最开始插入时的内容。

![image-20220720160822920](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720160822920.png)

因此，我们可以看出对于改变数据列大小的更新，采用删除再插入的方式，并且该删除是直接删除，将更新后的数据添加到ibd数据页中，并且产生undo日志记录，记录的是修改前的内容。

#### 4. 回滚

最后我们将该事务回滚掉，rollback

然后再查看下ibd文件数据页内容，可以看到数据内容回滚到了事务前的内容，并且存放的位置是我们新插入的那个地方。

![image-20220720161255599](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720161255599.png)

#### 补充

在最后一步回滚中由于新插入的数据是在最后的，无法分辨出是删除再添加还是直接原地更新。因此验证下最后的回滚操作。

1. 重建表和数据

```mysql
drop table test_undo;
create table test_undo(
    id int,
    name varchar(10),
    primary key (id)
)engine=innodb charset=utf8 row_format=dynamic;
insert into test_undo values (1,'aaa'),(2,'xxx');
```

目前ibd文件

```shell
00010060  02 00 1c 69 6e 66 69 6d  75 6d 00 03 00 0b 00 00  |...infimum......|
00010070  73 75 70 72 65 6d 75 6d  03 00 00 00 10 00 1b 80  |supremum........|
00010080  00 00 01 00 00 00 00 55  2e 81 00 00 00 9d 01 10  |.......U........|
00010090  61 61 61 03 00 00 00 18  ff d6 80 00 00 02 00 00  |aaa.............|
000100a0  00 00 55 2e 81 00 00 00  9d 01 1d 78 78 78 00 00  |..U........xxx..|
000100b0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```

2. 执行更新操作，并且先不回滚

```mysql
set autocommit=0;
begin;
update test_undo set name='ccccc' where id=1;
```

目前ibd文件

```shell
00010060  02 00 52 69 6e 66 69 6d  75 6d 00 03 00 0b 00 00  |..Rinfimum......|
00010070  73 75 70 72 65 6d 75 6d  03 00 00 00 10 00 00 80  |supremum........|
00010080  00 00 01 00 00 00 00 55  2e 81 00 00 00 9d 01 10  |.......U........|
00010090  61 61 61 03 00 00 00 18  ff d6 80 00 00 02 00 00  |aaa.............|
000100a0  00 00 55 2e 81 00 00 00  9d 01 1d 78 78 78 05 00  |..U........xxx..|
000100b0  00 00 20 ff e5 80 00 00  01 00 00 00 00 55 33 02  |.. ..........U3.|
000100c0  00 00 02 33 07 ca 63 63  63 63 63 00 00 00 00 00  |...3..ccccc.....|
000100d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```

3. 填充末尾

由于第一条是空闲空间，因此需要添加两条数据

```mysql
insert into test_undo values (3,'yyy'),(4,'zzz');
```

目前ibd文件

```shell
00010060  02 00 52 69 6e 66 69 6d  75 6d 00 05 00 0b 00 00  |..Rinfimum......|
00010070  73 75 70 72 65 6d 75 6d  03 00 00 00 10 00 53 80  |supremum......S.|
00010080  00 00 03 00 00 00 00 55  34 81 00 00 00 9f 01 10  |.......U4.......|
00010090  79 79 79 03 00 00 00 18  ff e5 80 00 00 02 00 00  |yyy.............|
000100a0  00 00 55 2e 81 00 00 00  9d 01 1d 78 78 78 05 00  |..U........xxx..|
000100b0  00 00 20 ff e5 80 00 00  01 00 00 00 00 55 33 02  |.. ..........U3.|
000100c0  00 00 02 33 07 ca 63 63  63 63 63 03 00 00 00 28  |...3..ccccc....(|
000100d0  ff 9e 80 00 00 04 00 00  00 00 55 34 81 00 00 00  |..........U4....|
000100e0  9f 01 1d 7a 7a 7a 00 00  00 00 00 00 00 00 00 00  |...zzz..........|
000100f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```

4. 回滚更新

目前ibd文件

```shell
00010060  02 00 52 69 6e 66 69 6d  75 6d 00 05 00 0b 00 00  |..Rinfimum......|
00010070  73 75 70 72 65 6d 75 6d  03 00 00 00 10 00 53 80  |supremum......S.|
00010080  00 00 03 00 00 00 00 55  34 81 00 00 00 9f 01 10  |.......U4.......|
00010090  79 79 79 03 00 00 00 18  ff e5 80 00 00 02 00 00  |yyy.............|
000100a0  00 00 55 2e 81 00 00 00  9d 01 1d 78 78 78 03 00  |..U........xxx..|
000100b0  00 00 20 ff e5 80 00 00  01 00 00 00 00 55 2e 81  |.. ..........U..|
000100c0  00 00 00 9d 01 10 61 61  61 63 63 03 00 00 00 28  |......aaacc....(|
000100d0  ff 9e 80 00 00 04 00 00  00 00 55 34 81 00 00 00  |..........U4....|
000100e0  9f 01 1d 7a 7a 7a 00 00  00 00 00 00 00 00 00 00  |...zzz..........|
000100f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```

5. 结论：可以看到回滚的操作是在新插入的数据上做更新的

### 三、测试删除再插入2

根据上一步测试我们知道更新的数据大于我们当前的数据，那么会先删除再插入，那我们再来看下如果更新的数据是小于的呢。

我们将表删除再重新构建下。

```mysql
drop table test_undo;

create table test_undo(
    id int,
    name varchar(10),
    primary key (id)
)engine=innodb charset=utf8 row_format=dynamic;

insert into test_undo values (1,'aaa'),(2,'xxx');
```

重新查看table_id

```mysql
mysql> select TABLE_ID from information_schema.INNODB_TABLES where name like '%test_undo%';
+----------+
| TABLE_ID |
+----------+
|     1273 |
+----------+
1 row in set (0.01 sec)
```

#### 1. 查看ibd

可以看到该数据行的trx_id为0x54f8，即21752

![image-20220720163632472](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720163632472.png)

再查看一下page_free和page_garbage字段的值

![image-20220720163816978](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720163816978.png)

#### 2. 修改数据

```mysql
set autocommit=0;
begin;
update test_undo set name='dd' where id=1;
```

开启事务修改内容，注意这里修改的大小是比原数据小的。同样先不要提交和回滚，先卡在这里。

接着查看该事务的事务id，为21757，即0x54fd

```mysql
mysql> select trx_id from information_schema.INNODB_TRX;
+--------+
| trx_id |
+--------+
|  21757 |
+--------+
1 row in set (0.01 sec)
```

#### 3. 查看ibd以及undo日志

接下来查看ibd文件数据页内容，我们看到该行数据是在原数据行上变更的，变更为了'bbb'，并且trx_id为0x54fd，也就是我们目前正在执行的事务的id。

![image-20220720164043135](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720164043135.png)

接下来查看undo日志，可以看到该undo日志记录的是该更新前的数据的内容。

![image-20220720164342652](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720164342652.png)

#### 4. 回滚

接下来我们将该事务回滚掉，rollback。还记得回滚前ibd文件长啥样吗？往上看看。

接下来我们查看回滚后的ibd文件，发现居然在后面新添加了一条数据，存放的内容就是更新前的内容。

![image-20220720164717699](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720164717699.png)

那我们再来看看page_free和page_garbage字段，发现这俩字段不为0了，说明将第一条记录删除掉了。

![image-20220720164912423](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20220720164912423.png)

#### 5. 提交

如果这里最后选择的是commit而不是rollback的话，那么就是直接在原数据行上应用更新了，并不会插入新的数据行。

### 总结

针对update操作的undo日志，在不更新主键的情况下，并且更新的列非索引：

- 如果更新的列占用的空间未发生变化，则就地更新，在原数据行上做更新，并且回滚后还是在该数据行
- 如果更新的列占用的空间比原来占用的空间大
  - 在更新操作后，直接删除原数据，并且插入更新后的新数据行
  - 回滚操作，则是在新数据行进行更新
- 如果更新的列占用的空间比原来占用的空间小
  - 在更新操作后，首先会在原数据行上直接做更新
  - 如果对该更新操作回滚，那么会删除原数据行，并且插入更新前的数据行

