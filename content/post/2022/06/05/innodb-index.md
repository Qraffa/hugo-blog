---
title: "InnoDB索引"
date: 2022-06-05T10:11:32+08:00
url: post/2022/06/05/innodb-index
tags:
- mysql
- innodb
categories:
- mysql
---

## InnoDB索引

### 单机InnoDB

#### InnoDB磁盘相关介绍

![](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/innodb-architecture.png)

<center>InnoDB-架构图</center>

InnoDB存储引擎是基于磁盘存储的，并将其中的记录按页的方式管理。InnoDB在内存中创建一个缓冲池，当进行读取页操作时，首先将从磁盘读到的页存放在缓冲池中。下一次再读到相同的页时，首先判断该页是否在缓冲池中。若在缓冲池中，则在缓冲池中直接读取该页即可；否则，读取磁盘上的该页。当进行修改操作时，首先修改在缓冲池中的页，然后再以一定的频率刷新到磁盘上。页从缓冲池中被刷回磁盘的操作并不是在每次页发生更新时触发，而是通过checkpoint的机制触发刷回。

#### InnoDB表空间结构

![](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/tablespace_layout.png)

<center>InnoDB-逻辑存储结构</center>

在innodb存储引擎中，表都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表。在innodb存储引擎中，每张表都有个主键，如果在创建表时没有显示地定义主键，则innodb存储引擎会使用表中非空的唯一索引或自动创建一个6字节大小的指针。

innodb存储引擎中，所有数据都被逻辑地存放在表空间（tablespace）中，表空间由段（segment）、区（extent）、页（page）组成。

表空间只存放数据、索引和插入缓冲bitmap页，其他比如回滚信息、插入缓冲索引页等存放在共享表空间中。

段包括数据段、索引段、回滚段等。数据段即为B+树的叶子节点，索引段即为B+树的非叶子节点。

区是由连续页组成的空间，每个区的大小都为1MB。默认情况下，innodb存储引擎页的大小为16KB，即一个区有64个连续的页。

页是innodb磁盘管理的最小单位，常见有数据页、undo页等。

##### ibd文件分析

![IBD_File_Overview](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/IBD_File_Overview.webp)

<center>表文件结构</center>

###### FSP_HDR/XDES

在idb文件的第一个页的类型为**FSP_HDR**，该page主要用于存储表空间的一些关键信息。

![](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/FSP_HDR_Page_Overview.webp)

<center>FSP_HDR结构</center>

与其他所有页一样，FSP_HDR也是页，因此同样存在页头FIL_HEADER以及页尾FIL_TRAILER

###### FIL Header/Trailer

![](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/FIL_Header_and_Trailer.webp)

<center>页头、页尾结构</center>

| 名称(fil0types.h)                | 含义                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| FIL_PAGE_SPACE_OR_CHKSUM         | 表示页的校验和                                               |
| FIL_PAGE_OFFSET                  | 页号                                                         |
| FIL_PAGE_PREV                    | 逻辑上前一个页.在page0中，该字段为FIL_PAGE_SRV_VERSION       |
| FIL_PAGE_NEXT                    | 逻辑上后一个页.在page0中，该字段为FIL_PAGE_SPACE_VERSION     |
| FIL_PAGE_LSN                     | 该页最后被修改时的LSN                                        |
| FIL_PAGE_TYPE                    | 页类型                                                       |
| FIL_PAGE_FILE_FLUSH_LSN          | 仅在系统表空间的第1页定义，代表文件至少被刷新到该LSN值       |
| FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID | 页所在表空间id                                               |
| FIL_PAGE_END_LSN_OLD_CHKSUM      | 前4个字节表示检验和，后4个字节应当与FIL_PAGE_LSN后4个字节相同 |

###### FSP Header

![FSP_Header](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/FSP_Header.webp)

<center>FSP Header结构</center>

| 名称(fsp0fsp.h)      | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| FSP_SPACE_ID         | 表空间id                                                     |
| FSP_SIZE             | 当前表空间拥有的页面数                                       |
| FSP_FREE_LIMIT（？） | 尚未被初始化的最小页号,大于等于该页号的区对应的XDES Entry结构都没有加入到FREE链表中.在小于64个页的表空间中，该值为64 |
| FSP_SPACE_FLAGS      | 存储标志                                                     |
| FSP_FRAG_N_USED      | FSP_FREE_FRAG链表中已使用的页面数量                          |
| FSP_FREE             | 空闲extents的基节点，该链上的extent所有page均未被使用，可以整个extent分配给segment，或使用部分碎片页并移动到FSP_FREE_FRAG链 |
| FSP_FREE_FRAG        | FREE_FRAG链表的基节点，该链上的extent存在空闲页，通常这样的extent中的page，会分配到不同的segment，挂在segment的FSEG_FRAG_ARR上。比如每个带了FSP_HDR页或XDES页的extent就是FSP_FREE_FRAG的 |
| FSP_FULL_FRAG        | FULL_FRAG链表的基节点，该链上的extent没有空闲page，当该extent上有page被释放后，可以移回FSP_FREE_FRAG链 |
| FSP_SEG_ID           | 当前表空间中下一个未使用的SegementID,即下次待分配的SegmentID |
| FSP_SEG_INODES_FULL  | SEG_INODES_FULL链表的基节点，该链上的INODE页已经没有空闲INODE Entry了，一个INODE页最多存放85个INODE Entry。在独立表空间中，当为一个表创建超过42个索引，才会出现FULL的INODE页 |
| FSP_SEG_INODES_FREE  | SEG_INODES_FREE链表的基节点，该链上的INODE页存在空闲的INODE Entry |

###### XDES Entry

XDES 指一个page

XDES Entry 这个page里的一段数据，管理一个extent

![XDES_Entry](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/XDES_Entry.webp)

<center>XDES Entry结构</center>

| 名称           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| XDES_ID        | 如果该extent属于某个segment，则为该extent所属的segment的segment ID |
| XDES_FLST_NODE | XDES Entry链表的前后指针                                     |
| XDES_STATE     | 该extent的状态                                               |
| XDES_BITMAP    | 共128位，表示该extent中64个页的状态，每个页2位，第1位表示是否空闲，第2位未使用 |

extent的几种状态

| 状态             | 含义                                                  |
| ---------------- | ----------------------------------------------------- |
| XDES_FREE=1      | 空闲区，存在于表空间管理的FREE链表上                  |
| XDES_FREE_FRAG=2 | 有空闲页面的碎片区，存在于表空间管理的FREE_FRAG链表上 |
| XDES_FULL_FRAG=3 | 无空闲页面的碎片区，存在于表空间管理的FULL_FRAG链表上 |
| XDES_FSEG=4      | 归属于ID为XDES_ID的段的区                             |
| XDES_FSEG_FRAG=6 | 暂时给段使用的碎片区，见下文segment分配extent         |

###### IBUF_BITMAP

可以看到在ibd文件中第二个页是IBUF_BITMAP，与change buffer相关，本文先不展开讲

每个分组中第二个页面都是IBUF_BITMAP页。第一个extent的第一个页是FPS_HDR，其他的都是XDES页，XDES页与FPS_HDR页除了没有FSP Header外其他相同。

![IBUF_BITMAP Page Overview](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/IBUF_BITMAP%20Page%20Overview.png)

<center>IBUF_BITMAP页结构</center>

![IBUF_BITMAP Entry](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/IBUF_BITMAP%20Entry.png)

<center>描述每个页的change buffer的结构</center>

###### INODE

第三个页面是INODE页，用于管理表空间的segment。

XDES/XDES Entry

INODE是一个page指管理里面的INODE

INODE Entry指的是一个segment

![INODE_Page_Overview](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/INODE_Page_Overview.webp)

<center>INODE页结构</center>

| 名称                 | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| FSEG_INODE_PAGE_NODE | INODE页链表节点，用于链接前后INODE页。如果一个表空间中的段数量超过85个，则一个INODE页不足以存储所有段信息，因此需要新的INODE页。BaseNode记录在头Page的FSP_SEG_INODES_FULL或者FSP_SEG_INODES_FREE字段 |
| INDEO Entry 0...84   | 存储段信息，一个INODE页，可以存储85个INODE Entry             |

![INODE_Entry](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/INODE_Entry.webp)

| 名称                 | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| FSEG_ID              | 该INODE Entry所属的segment ID.如果为0，表示该Entry未被使用   |
| FSEG_NOT_FULL_N_USED | FSEG_NOT_FULL链表已经使用的page的数量                        |
| FSEG_FREE            | 该segment上，完全没有被使用并分配给该Segment的Extent链表     |
| FSEG_NOT_FULL        | 该segment上，存在空闲页面的extent的链表                      |
| FSEG_FULL            | 该segment上，没有空闲页面的extent的链表                      |
| FSEG_MAGIC_N         | magic number                                                 |
| FSEG_FRAG_ARR        | 属于该segment的独立page，存储为页号。从全局的碎片区(FREE_FRAG或FULL_FRAG)分配来的页。当填满32个数组项后，之后每次分配都将分配整个extent，并且被分配的extent的XDES_ID为该entry的FSEG_ID |

###### 索引页

真正存储构建用户数据的页面

![INDEX_Page_Overview](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/INDEX_Page_Overview.webp)

<center>数据页结构</center>

首先是每个页都有的页头结构和页尾结构，不再重复。

**描述索引信息的INDEX Header**

| 名称             | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| PAGE_N_DIR_SLOTS | page directory中slot的个数                                   |
| PAGE_HEAP_TOP    | 指向当前page已使用的空间的末尾偏移，也即free space的开始位置 |
| PAGE_N_HEAP      | 第一位表示该页格式为Compact，剩余位表示page内所有记录的数量(包括infimum和supremum，以及标记删除的记录) |
| PAGE_FREE        | 表示标记删除的链表头节点的偏移。各个被标记删除的记录通过next_record组成单向链表，这些记录可以被重新使用 |
| PAGE_GARBAGE     | 标记删除链表占用的字节数                                     |
| PAGE_LAST_INSERT | 最近一次插入的记录的偏移                                     |
| PAGE_DIRECTION   | 最近一次插入的方向，每次插入时，PAGE_LAST_INSERT会和当前记录进行比较，以确认插入方向 |
| PAGE_N_DIRECTION | 一个方向连续插入的记录数量                                   |
| PAGE_N_RECS      | 该页中记录数量，不包括infimum和supremum，以及标记删除的记录  |
| PAGE_MAX_TRX_ID  | 修改当前页的最大事务id，仅在二级索引和insert buffer tree中定义 |
| PAGE_LEVEL       | 该页所在b+树的层级，叶子节点为0，根节点level最大             |
| PAGE_INDEX_ID    | 该页归属的索引ID                                             |

**描述段信息的FSEG Header**

一个索引会产生两个segment，分别为leaf node segment和non-leaf node segment，而每个segment会对应一个INODE Entry结构。因此在b+树的根节点页面中会有FSEG Header来描述该b+树的叶节点和非叶节点的segment相关信息。

| 名称              | 含义                              |
| ----------------- | --------------------------------- |
| PAGE_BTR_SEG_LEAF | b+树叶节点所在segment的头部信息   |
| PAGE_BTR_SEG_TOP  | b+树非叶节点所在segment的头部信息 |

![](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/FSEG_Header.webp)

<center>Segment Header结构</center>

| 名称             | 含义                               |
| ---------------- | ---------------------------------- |
| FSEG_HDR_SPACE   | 该segment的所在的INODE页的space id |
| FSEG_HDR_PAGE_NO | 该segment的所在的INODE页的page no  |
| FSEG_HDR_OFFSET  | inode entry在INODE页内的偏移       |

查看页面内容

```haskell
--- IBD PAGE #3 ---
FIL_PAGE_OFFSET: 3
FIL_PAGE_TYPE: 17855
PAGE_BTR_SEG_LEAF (for root only):
- FSEG_HDR_SPACE: 108
- FSEG_HDR_PAGE_NO: 2
- FSEG_HDR_OFFSET: 242
PAGE_BTR_SEG_TOP (for root only):
- FSEG_HDR_SPACE: 108
- FSEG_HDR_PAGE_NO: 2
- FSEG_HDR_OFFSET: 50
```

- 叶节点和非叶节点所在的segment的INODE Entry都在page#2中
- 叶节点和非叶节点所在的segment的INODE Entry所在的偏移分别为242和50，一个INODE Entry占用192bytes
- 再回看INODE页INODE Entry#0从页面偏移50开始，INODE Entry#1从页面偏移242开始

**System Records两个系统记录**，用于描述该页上最小最大的虚记录，在该页内所有用户记录都大于infimum，所有用户记录都小于supremum。

infimum记录next_record指向该页中用户记录中最小的记录。supremum记录的next_record指向0，表示没有后续。同时该页中用户记录最大的记录的next_record指向supremum记录。

接下来介绍用户记录结构时在详细解析各个字段含义

![INDEX_System_Records](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/INDEX_System_Records.webp)

<center>System Records结构</center>

```c++
/** The page infimum and supremum of an empty page in ROW_FORMAT=COMPACT */
static const byte infimum_supremum_compact[] = {
    /* the infimum record */
    0x01 /*n_owned=1*/, 0x00, 0x02 /* heap_no=0, REC_STATUS_INFIMUM */, 0x00,
    0x0d /* pointer to supremum */, 'i', 'n', 'f', 'i', 'm', 'u', 'm', 0,
    /* the supremum record */
    0x01 /*n_owned=1*/, 0x00, 0x0b /* heap_no=1, REC_STATUS_SUPREMUM */, 0x00,
    0x00 /* end of record list */, 's', 'u', 'p', 'r', 'e', 'm', 'u', 'm'};
```

**用户数据记录user records**

![Record_Header](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/Record_Header.webp)

<center>记录头信息结构</center>

| 名称              | 大小   | 含义                                                         |
| ----------------- | ------ | ------------------------------------------------------------ |
| 变长字段长度列表  |        |                                                              |
| NULL值列表        |        |                                                              |
| REC_NEW_INFO_BITS | 4bits  | 前两位未使用，一位为REC_INFO_MIN_REC_FLAG，b+树每层非叶节点中最小的记录会被设置为1；一位为REC_INFO_DELETED_FLAG标记该记录被删除 |
| REC_NEW_N_OWNED   | 4bits  | 该记录所在的slot含有的记录个数，该slot中最大的记录会设置该值，该slot中其他记录为该值为0 |
| REC_NEW_HEAP_NO   | 13bits | 该记录在页面堆中的位置，infimum记录为0，supremum记录为1，用户记录从2开始 |
| REC_NEW_STATUS    | 3bits  | REC_STATUS_ORDINARY=0表示叶节点普通记录，REC_STATUS_NODE_PTR=1表示非叶节点，REC_STATUS_INFIMUM=2表示infimum记录，REC_STATUS_SUPREMUM=3表示supremum记录 |
| REC_NEXT          | 2bytes | 表示下一条记录的数据偏移，存储的是和当前记录的相对位置偏移量 |

备注：REC_INFO_MIN_REC_FLAG：this bit is set if and only if the record is the first user record on a non-leaf B-tree page that is the leftmost page on its level

数据记录的链接如下，可以看到REC_NEXT指向的是下一个记录真实数据开始的地方

![0509](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/0509.png)



<center>next_record示例</center>

引用一张数据页格式总览

![b1ffab644960e13617606f2cc35d124c](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/b1ffab644960e13617606f2cc35d124c.png)

<center>数据页格式</center>

**空闲空间Free Space**，从最后一个用户记录(PAGE_HEAP_TOP)到最后一个page directory的空间为空闲空间。当用户需要插入记录时候，首先在被删除的记录的空间中查找，如果没有找到合适的空间，就从这里分配。

**page directory数据页目录**，在数据页中可能存在非常多的用户记录数据，并且这些数据是以链表有序组成的。那么当我们需要页中具体数据行时，如果通过遍历整个链表，那么效率将比较低下，所以对页内的数据行做个数据页目录，就可以使用二分，以提升查询效率。

![INDEX_Page_Directory](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/INDEX_Page_Directory.webp)

<center>page directory结构</center>

在page directory中每个slot占用2byte，表示该slot对应的数据行在页内的偏移。一个页在初始时就存在两个slot，分别是infimum和supremum。其中infimum只有infimum记录本身，因此其n_owned总是1；除了supremum没有没有最小值限制外，其他由用户插入数据记录而产生的新的slot，n_owned取值范围都在[4,8]

值得注意的是，一个slot对应的数据行，是该slot中所有数据行的最大记录。

```haskell
PAGE_N_DIR_SLOTS: 2
REC OFFSETS: 0 EXPECTED
	OFFSET TO INFIMUM: 99
		OFFSET: 99
		NULL BITS: 00000001
		INFO BITS (?|?|DELETED|MIN_REC): 0000
		N_OWNED: 1
		HEAP_NO: 0-PAGE_HEAP_NO_INFIMUM
		RECORD_TYPE: 010-Infimum
		NEXT OFFSET: 112
	OFFSET TO SUPREMUM: 112
		OFFSET: 112
		NULL BITS: 00000001
		INFO BITS (?|?|DELETED|MIN_REC): 0000
		N_OWNED: 1
		HEAP_NO: 1-PAGE_HEAP_NO_SUPREMUM
		RECORD_TYPE: 011-Supremum
		NEXT OFFSET: 112
PAGE_DIRECTORY: 2 SLOTS
	SLOT[0]: 99
	SLOT[1]: 112
```

page directory插入时维护：

1. 初始情况下只有infimum slot和supremum slot
2. 当插入新记录后，首先从page directory中查找到对应记录的主键值比待插入记录的主键值大并且差值最小的slot，然后将该slot的n_owned值+1，直到该slot o_owned数量等于8
3. 当一个slot中记录数等于8后，再插入一条记录，会将slot中的记录拆分成两个slot，其中一个slot中记录数为4，另一个为5

在数据页中查指定主键值得记录时：

1. 通过二分定位该记录所在的slot，然后通过slot的连续性，找到该slot的上一个slot‘
2. 从slot’对应的数据行记录开始，通过next_record遍历查找slot中的各个记录

![0515](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/0515.png)

<center>page directory示例</center>

##### 文件维护

InnoDB通过inode entry来管理每个Segment占用的数据页，每个segment可以看做一个文件页维护单元。inode entry所在的inode页有可能存放满，因此又通过FSP_HDR维护了Inode Page链表，以及各个inode页的通过FSEG_INODE_PAGE_NODE链接。

在ibd文件的第一个页FSP_HDR中还维护了表空间内extent的三个链表，分别为FREE、FREE_FRAG、FULL_FRAG；同时在inode entry中同样维护了extent的三个链表，分别为FREE、FSEG_NOT_FULL、FSEG_FULL。segment可以从表空间申请extent并添加到inode entry中的extent链表中，也可以申请页添加到自己的独立page数组中。

当创建一个新的索引时，实际构建一个新的btree(btr_create)，先为非叶节点申请inode entry，再创建root page(该page属于刚申请的segment)作为btree的根节点，并将申请到的segment的信息写入到root page的PAGE_BTR_SEG_TOP中；然后再为叶节点申请inode entry，并将申请到的segment的信息写入到root page的PAGE_BTR_SEG_LEAF中。

###### segment创建

函数：fseg_create_general()

主要流程：

1. fsp_alloc_seg_inode()向fsp申请一个inode entry
   1. 如果一个inode页的inode entry都用完了，则需要fsp_alloc_seg_inode_page()申请新的inode页
   2. 将新申请的inode页添加到FSP_SEG_INODES_FREE

1. 对申请到的inode entry做初始化
   1. 包括更新FSP_SEG_ID，初始化inode entry中的extent链表，初始化inode entry中的碎片页数组

1. 根据参数中的page_no，决定是否创建新page。如果参数中page_no为0，则会创建segment头的新page

1. 将此次创建的inode entry的头信息写入到上述的page中对应位置

总结：创建时可能需要对文件扩展保证足够空间，然后会向fsp申请inode entry，此时inode页又可能放满了inode entry，需要申请新的inode页。在拿到inode entry后，则进行初始化，然后将该inode的信息写入到page中。在b+tree创建时，会将叶段和非叶段写入到root page的FSEG Header中。

###### extent分配

**表空间分配**

函数：fsp_alloc_free_extent()

主要流程：

1. xdes_get_descriptor_with_space_hdr()获取hint page所在的extent的xdes entry
   1. 如果该xdes entry为XDES_FREE，则我们直接使用该extent返回

1. 尝试从fsp的FSP_FREE获取空闲的extent
   1. 如果不存在，则需要fsp_fill_free_list()扩展新的空间
   2. 扩展新extent时，如果设计xdes page的初始化，则会将这样的xdes添加到FSP_FREE_FRAG，其他则添加到FSP_FREE

1. 再次从fsp的FSP_FREE获取空闲的extent

1. 将该extent从FSP_FREE移除并返回

**段分配**

函数：fseg_alloc_free_extent()

主要流程：

1. 尝试从segment的FSEG_FREE上获取extent

1. fsp_alloc_xdes_free_frag()尝试从fsp的FSP_FREE_FRAG获取
   1. fsp_get_last_free_frag_extent()获取FSP_FREE_FRAG链表的最后一个extent
   2. xdes_is_leasable()检查该extent能否分配给segment
      1. 要求该extent的前两个页被使用且其他页都是空闲的，其实就是一组extent最开始的那个extent，第一个用于XDES，第二个页用于IBUF_BITMAP
   3. 能够分配，则将extent从FSP_FREE_FRAG移除，设置seg_id，设置状态为XDES_FSEG_FRAG，并添加到segment的FSEG_NOT_FULL，直接返回该extent

1. fsp_alloc_free_extent()向fsp申请extent
   1. 从fsp申请来的extent，设置seg_id，设置状态为XDES_FSEG，并添加到segment的FSEG_FREE

###### page分配

**表空间分配**

函数：fsp_alloc_free_page()

主要流程：

1. 获取hint页所在的extent，如果该extent是XDES_FREE_FRAG，则我们使用该extent

1. 否则从FSP_FREE_FRAG中获取extent
   1. 从FSP_FREE_FRAG获取不到时，需要fsp_alloc_free_extent()申请空闲的extent，并添加到FSP_FREE_FRAG中

1. 从获得的extent中，xdes_find_bit()查找空闲page

1. fsp_alloc_from_free_frag()分配该page
   1. 设置xdes的bitmap，表示该page被使用。更新fsp的FSP_FRAG_N_USED
   2. 如果该extent满了，还需要从FSP_FREE_FRAG中移除，添加到FSP_FULL_FRAG中

1. fsp_page_create()初始化该page，并返回

**段分配**

函数：fseg_alloc_free_page_low()

主要流程：

1. fseg_n_reserved_pages_low()统计该segment占用和使用的page数
   1. 占用：统计FSEG_FULL、FSEG_NOT_FULL、FSEG_FREE的总page数以及frag pages
   2. 使用：统计FSEG_FULL、FSEG_NOT_FULL的已使用的page数以及frag pages

1. xdes_get_descriptor_with_space_hdr()获取hint page所在的extent的xdes
   1. 如果hint page超过限制，会将hint reset成0，重新获取xdes_get_descriptor()

1. 如果
   1. **take_hinted_page**
   2. 条件：
      1. 该extent状态为XDES_FSEG/XDES_FSEG_FRAG，
      2. 并且属于要求的segment，
      3. 并且hint page在extent中是空闲的，
   3. 则**got_hinted_page**

1. 如果
   1. 条件：
      1. 该extent状态为XDES_FREE
      2. 并且使用page数大于总占用的7/8
      3. 并且使用page数大于32
   2. 则fsp_alloc_free_extent()从fsp申请extent，然后**take_hinted_page**

1. 如果
   1. 条件：
      1. direction != FSP_NO_DIR
      2. 并且使用page数大于总占用的7/8
      3. 并且使用page数大于32
   2. 则fsp_alloc_free_extent()从fsp申请extent
   3. 如果direction = FSP_UP
      1. 如果该extent状态为XDES_FSEG_FRAG，则需要xdes_find_bit()找空闲页
      2. 否则直接取第一个页
   4. 如果direction = FSP_DOWN
      1. 则取extent的最后一个页

1. 如果
   1. 条件：
      1. 该extent属于该segment
      2. 并且该extent存在空闲页
   2. 则从该extent中xdes_find_bit()找空闲页

1. 如果
   1. 条件：
      1. 该segmnet占用的page数大于已使用的page数，则说明该segment存在空闲page
   2. 则依次检查FSEG_NOT_FULL和FSEG_FREE，获取有空闲页的extent
   3. 然后从取到的extent中xdes_find_bit()找空闲页

1. 如果
   1. 条件：
      1. 该segment中已使用的page数小于32
   2. 则fsp_alloc_free_page()从表空间获取page
   3. 然后fseg_find_free_frag_page_slot()找到该segment的空闲碎片slot
   4. 并fseg_set_nth_frag_page_no()将获取到的page放入到空闲位置

1. 当以上全都不满足时
   1. 则fseg_alloc_free_extent()为该segmnet分配一个空闲的extent
   2. 如果分配的extent状态为XDES_FSEG_FRAG，则需要xdes_find_bit()找空闲页
   3. 否则直接取第一个页

1. 对获取的page做一些检查

1. **got_hinted_page**
   1. fseg_mark_page_used()标记extent中该page被使用
      1. 该方法只有当我们是从fsp_alloc_free_page()分配，也即上面第8点分配时，才不需要做这一步
   2. fsp_page_create()初始化该page，并返回

总结：

上述流程比较复杂，主要是优先填满32个碎片页，之后才会分配完整的extent，会尽可能获得hint page，当segment上未使用的page较多时，会优先从segment寻找空闲page

##### 整体结构

![2](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/2.png)

<center>InnoDB 表空间page管理</center>

![0903](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/0903.png)

<center>每个组中头几个页面的类型</center>

##### btree

###### btree创建

函数：btr_create()

备注：忽略ibuf tree，以下只分析index tree

主要流程：

1. fseg_create()创建non-leaf segment。注意此时page参数为0，参考上面segment创建部分。
   1. 在此方法中，会创建一个segment和一个root page，并将创建的segment的信息写入到root page的PAGE_BTR_SEG_TOP中

1. fseg_create()创建leaf segment。注意此时page参数为上一步返回的page_no
   1. 在此方法中，会创建一个segment，并将创建的segment的信息写入到root page的PAGE_BTR_SEG_LEAF中

1. page_create()对root page做一些初始化操作

1. btr_page_set_level()设置root page的level为0，即PAGE_LEVEL

1. btr_page_set_index_id()设置root page的index_id，即PAGE_INDEX_ID

1. btr_page_set_next()、btr_page_set_prev()设置root page的前后页为FIL_NULL

###### b+树检索

1. btr_cur_search_to_nth_level()从root  page向下查找，直到我们指定的level，将btree cursor定位在第一个大于/小于(等于)tuple的位置
   1. page_cur_search_with_match()对指定Page的Page Directory(页目录)进行二分查找定位Record周边的两个slot
   2. 从low_rec开始线性迭代直到up_rec，查找符合条件的Record

###### b+树维护

**record插入**

1. 先尝试乐观插入btr_cur_optimistic_insert()
   1. 通过 cursor 定位 leaf page, 计算 record 的物理长度
   2. 如果page空间足够，则page_cur_insert_rec_low()插入记录
   3. 如果page空间不足够插入该记录，则乐观插入失败
2. 乐观插入失败，则悲观插入btr_cur_pessimistic_insert()
   1. fsp_reserve_free_extents()确保有足够空闲空间插入记录
   2. 如果是root page，则btr_root_raise_and_insert()
   3. 其他，则btr_page_split_and_insert()，btree页分裂，并插入记录

值得注意的是，一个btree的索引一旦创建，那么他的root page对应的page no将不会再变更。所有索引root page信息都存储在系统表SYS_INDEXES中。

![btr_page_split_and_insert](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/btr_page_split_and_insert.png)

<center>节点分裂流程</center>

![btr_root_raise_and_insert](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/btr_root_raise_and_insert.png)

<center>树层加高流程</center>

## 参考

[MySQL · 引擎特性 · Innodb 表空间](http://mysql.taobao.org/monthly/2019/10/01/)

[MySQL · 引擎特性 · InnoDB 数据页解析](http://mysql.taobao.org/monthly/2018/04/03/)

[MySQL · 数据恢复 · undrop-for-innodb](http://mysql.taobao.org/monthly/2017/11/01/)

[MySQL · 引擎特性 · InnoDB Buffer Pool](http://mysql.taobao.org/monthly/2017/05/01/)

[MySQL · 引擎特性 · InnoDB 文件系统之文件物理结构](http://mysql.taobao.org/monthly/2016/02/01/)

[MySQL · 引擎特性 · InnoDB 文件系统之IO系统和内存管理](http://mysql.taobao.org/monthly/2016/02/02/)

[MySQL · 引擎特性 · InnoDB IO子系统](http://mysql.taobao.org/monthly/2017/03/01/)

[MySQL · 引擎特性 · 手动分析InnoDB B+Tree结构](http://mysql.taobao.org/monthly/2020/04/06/)

[MySQL · 引擎特性 · InnoDB 数据文件简述](http://mysql.taobao.org/monthly/2020/08/06/)

[MySQL · 周边工具 · MySQL InnoDB inno_space 工具介绍](http://mysql.taobao.org/monthly/2021/11/02/)

[https://blog.jcole.us/innodb/](https://blog.jcole.us/innodb/)

[https://dev.mysql.com/doc/refman/8.0/en/innodb-architecture.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-architecture.html)

[深度 | 解析InnoDB引擎](https://cloud.tencent.com/developer/article/1509825)

[《MySQL是怎样运行的》](https://book.douban.com/subject/35231266/)

[《MySQL技术内幕》](https://book.douban.com/subject/24708143/)

[MySQL内核解析：Innodb页面存储结构-1](https://www.cnblogs.com/vinchen/archive/2012/09/10/2679478.html)

[InnoDB 的文件组织结构](https://leviathan.vip/2019/04/18/InnoDB%E7%9A%84%E6%96%87%E4%BB%B6%E7%BB%84%E7%BB%87%E7%BB%93%E6%9E%84/)

[InnoDB 中的 B+ 树的增删改](https://leviathan.vip/2020/07/18/innodb-b-plus-tree-curd/)

[InnoDB——Btree与MTR的牵扯](http://liuyangming.tech/05-2019/InnoDB-Mtr.html#btree%E6%82%B2%E8%A7%82%E6%8F%92%E5%85%A5%E6%B6%89%E5%8F%8A%E7%9A%84mtr)

[MySQL · 源码分析 · btr_cur_search_to_nth_level 函数分析](http://mysql.taobao.org/monthly/2021/07/02/)

[MySQL数据查询流程](http://blog.wudan3551.top/2021/03/27/mysql_query_in_deepth/)