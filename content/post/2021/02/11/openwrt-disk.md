---
title: "Openwrt Disk"
date: 2021-02-11T16:24:34+08:00
tags:
- openwrt
categories: 
- openwrt
---

## 树莓派openwrt，挂载u盘

### 参考

[https://blog.csdn.net/xjtumengfanbin/article/details/106871591](https://blog.csdn.net/xjtumengfanbin/article/details/106871591)

### 过程

- `fdisk /dev/mmcblk0`查看磁盘

- 输入`p`查看分区情况

  ```bash
  Device         Boot  Start      End  Sectors  Size Id Type
  /dev/mmcblk0p1 *      8192    49151    40960   20M  c W95 FAT32 (LBA)
  /dev/mmcblk0p2       57344   581631   524288  256M 83 Linux
  /dev/mmcblk0p3      581632 15269887 14688256    7G 83 Linux
  ```

- 输入`n`新建磁盘

- 选择`p`，然后选择`3`

- 在`First sector`中输入上一块磁盘的结束地址`/dev/mmcblk0p2   End`

- 在`Last sector`中输入默认值

- 输入`w`保存

- 执行`mkfs.ext4 /dev/mmcblk0p3`将新建的分区格式化

- 执行`mkdir /opt`新建目录，用于挂载

- 执行`mount -v -t ext4 -o rw /dev/mmcblk0p3 /opt`挂载

- Over

