---
title: MySql索引
date: 2020-04-29 13:38:56
tags:
- mysql
categories:
- mysql
---


## MySql索引

### 使用索引的优缺点

优点:

- 大大加快了数据的检索时间.
- 将随机IO变为顺序IO

缺点:

- 创建索引和维护索引耗费时间,对数据的增删改同时也需要动态维护索引
- 索引需要物理存储空间

<!--more-->

### 索引数据结构

1. Hash索引
   - 可以直接做hash,O(1)的复杂度查找
   - 无法做到范围查找和顺序查找
   - hash冲突会降低查找效率
2. B+树索引
   - 有序,可以做范围查找和顺序查找

### 索引类型

1. 主键索引

   数据库表中的主键使用主键索引.不能出现null,也不允许重复.

2. 唯一索引

   唯一索引的列不允许重复,但可以为null

3. 普通索引

   最基本的索引,允许重复和null

4. 组合索引

   多个字段创建的索引,遵循`最左前缀原则`,查询只使用复合索引最左侧的部分

5. 全文索引

   全文索引主要是为了检索大文本数据中的关键字的信息

### 聚集索引与非聚集索引

#### 聚集索引

索引结构和数据存在在一起,索引的顺序就是数据库物理存储的顺序,因此一个表只能有一个聚集索引.叶子结点存储了索引和索引对应的数据.

- 聚集索引查找速度快
- 依赖索引要有序
- 更新的代价大

#### 非聚集索引

索引结构和数据分开存放的索引,非聚集索引的索引在逻辑上连续,物理存储上不连续.叶子结点存储索引和索引对应数据的**指针**.

- 更新代价小
- 依赖索引要有序
- 可能需要进行回表查询,当查到索引对应的指针或主键后，可能还需要根据指针或主键再到数据文件或表中查询。

### 覆盖索引

如果一个索引包含（或者说覆盖）所有需要查询的字段的值，我们就称之为“覆盖索引”。

对于非聚集索引,需要根据查询的主键再进行一次回表查询,那么可以建立覆盖索引,将需要的数据包含在索引中,因此查询到索引后直接返回索引中的数据即可,不需要进行回表查询.

例如:

```sql
select col1, col2 from t1 where col1 = '213';
```

对于以上的查询,如果建立索引(col1,col2),那么查询之后不需要进行回表查询

### 创建索引的注意

#### 合适的字段建立索引

1. **不为null的字段**

   索引字段的数据应该尽量不为NULL，因为对于数据为NULL的字段，数据库较难优化。

2. **被频繁查询的字段**

   我们创建索引的字段应该是查询操作非常频繁的字段。

3. **被作为条件查询的字段**

   被作为WHERE条件查询的字段，应该被考虑建立索引。

4. **被频繁用于连接的字段**

   提高多表联合查询的效率

#### 不合适的字段

1. **被频繁更新的字段**

   索引的维护更新需要成本

2. **不被经常查询的字段**

   不必要建立索引

3. **尽量考虑联合索引而不是单列索引**

   索引是需要物理空间的,当表中的字段过多,建立了过多的单列索引会影响空间.

4. **注意避免冗余索引**

   两个索引覆盖的列相同就是冗余索引,例如(name,city)和(name).因此应该尽量扩展已有的索引而不是创建新的索引

5. **考虑对字符串使用前缀索引代替普通索引**

### 添加索引的操作

1.添加PRIMARY KEY（主键索引）

```
ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` )
```

2.添加UNIQUE(唯一索引)

```
ALTER TABLE `table_name` ADD UNIQUE ( `column` )
```

3.添加INDEX(普通索引)

```
ALTER TABLE `table_name` ADD INDEX index_name ( `column` )
```

4.添加FULLTEXT(全文索引)

```
ALTER TABLE `table_name` ADD FULLTEXT ( `column`)
```

5.添加多列索引

```
ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )
```