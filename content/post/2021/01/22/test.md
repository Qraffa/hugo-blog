---
title: "Test"
date: 2021-01-22T01:46:52+08:00
tags:
categories: 
---

## 基于Openwrt系统计并实现一个智能定位考勤系统

对基于Linux内核的开源OpenWrt系统进行分析，并能在常用路由器硬件开发平台上配置和执行，并基于此平台进行脚本语言编程设计，实现一个室内智能定位考勤系统。

### 摘要

### 第一章 绪论

1. 研究背景
2. 研究现状
3. 研究意义
4. 研究内容

### 第二章 相关技术介绍

1. WiFi技术
2. OpenWRT技术
3. 基于WiFi的距离估测

### 第三章 WiFi签到系统的设计与实现

1. wifi签到的原理流程
   1. 获取签到信号
   2. 距离估测
   3. 考勤信息上传服务端
2. 服务端考勤数据统计管理

### 第四章 系统测试



---

### 获取wifi信号的一些方法

- 查看wifi热点设备连接情况：

  - 方法一

  ```bash
  sudo arp
  
  地址                     类型    硬件地址            标志  Mask            接口
  192.168.123.1                    (incomplete)                              br-6e596f9c8376
  10.42.0.24               ether   24:31:54:a8:8b:92   C                     wlp3s0
  _gateway                 ether   00:74:9c:80:dc:7d   C                     enp4s0
  10.42.0.153              ether   80:ad:16:f7:89:f3   C                     wlp3s0
  ```

  - 方法二

  ```bash
  cat /proc/net/arp
  
  qraffa@qraffa:~$ cat /proc/net/arp 
  IP address       HW type     Flags       HW address            Mask     Device
  10.42.0.153      0x1         0x2         80:ad:16:f7:89:f3     *        wlp3s0
  192.168.31.1     0x1         0x2         28:d1:27:b7:ee:ea     *        enp4s0
  192.168.31.49    0x1         0x2         14:dd:a9:78:33:ac     *        enp4s0
  ```

  - 方法三

  ```bash
  cat /tmp/dhcp.leases
  ```

  - 方法四

  ```bash
  sudo iw dev wlp3s0 station dump
  
  Station 80:ad:16:f7:89:f3 (on wlp3s0)
  	inactive time:	48 ms
  	rx bytes:	283689
  	rx packets:	2562
  	tx bytes:	254734
  	tx packets:	1020
  	tx retries:	0
  	tx failed:	0
  	rx drop misc:	0
  	signal:  	-34 [-34, -73] dBm
  	signal avg:	-37 [-37, -73] dBm
  	tx bitrate:	1.0 MBit/s
  	rx bitrate:	72.2 MBit/s MCS 7 short GI
  	rx duration:	0 us
  	authorized:	yes
  	authenticated:	yes
  	associated:	yes
  	preamble:	long
  	WMM/WME:	yes
  	MFP:		no
  	TDLS peer:	no
  	DTIM period:	2
  	beacon interval:100
  	short slot time:yes
  	connected time:	83 seconds
  Station 24:31:54:a8:8b:92 (on wlp3s0)
  	inactive time:	0 ms
  	rx bytes:	120497
  	rx packets:	622
  	tx bytes:	73109
  	tx packets:	569
  	tx retries:	0
  	tx failed:	0
  	rx drop misc:	0
  	signal:  	-64 [-64, -73] dBm
  	signal avg:	-57 [-57, -73] dBm
  	tx bitrate:	1.0 MBit/s
  	rx bitrate:	72.2 MBit/s MCS 7 short GI
  	rx duration:	0 us
  	authorized:	yes
  	authenticated:	yes
  	associated:	yes
  	preamble:	long
  	WMM/WME:	yes
  	MFP:		no
  	TDLS peer:	no
  	DTIM period:	2
  	beacon interval:100
  	short slot time:yes
  	connected time:	8 seconds
  ```

  对于字段的解释

  `https://git.kernel.org/pub/scm/linux/kernel/git/linville/wireless.git/tree/include/uapi/linux/nl80211.h?id=HEAD`

  ```c
  /**
   * enum nl80211_sta_info - station information
   *
   * These attribute types are used with %NL80211_ATTR_STA_INFO
   * when getting information about a station.
   *
   * @__NL80211_STA_INFO_INVALID: attribute number 0 is reserved
   * @NL80211_STA_INFO_INACTIVE_TIME: time since last activity (u32, msecs)
   * @NL80211_STA_INFO_RX_BYTES: total received bytes (u32, from this station)
   * @NL80211_STA_INFO_TX_BYTES: total transmitted bytes (u32, to this station)
   * @NL80211_STA_INFO_RX_BYTES64: total received bytes (u64, from this station)
   * @NL80211_STA_INFO_TX_BYTES64: total transmitted bytes (u64, to this station)
   * @NL80211_STA_INFO_SIGNAL: signal strength of last received PPDU (u8, dBm)
   * @NL80211_STA_INFO_TX_BITRATE: current unicast tx rate, nested attribute
   * 	containing info as possible, see &enum nl80211_rate_info
   * @NL80211_STA_INFO_RX_PACKETS: total received packet (u32, from this station)
   * @NL80211_STA_INFO_TX_PACKETS: total transmitted packets (u32, to this
   *	station)
   * @NL80211_STA_INFO_TX_RETRIES: total retries (u32, to this station)
   * @NL80211_STA_INFO_TX_FAILED: total failed packets (u32, to this station)
   * @NL80211_STA_INFO_SIGNAL_AVG: signal strength average (u8, dBm)
   * @NL80211_STA_INFO_LLID: the station's mesh LLID
   * @NL80211_STA_INFO_PLID: the station's mesh PLID
   * @NL80211_STA_INFO_PLINK_STATE: peer link state for the station
   *	(see %enum nl80211_plink_state)
   * @NL80211_STA_INFO_RX_BITRATE: last unicast data frame rx rate, nested
   *	attribute, like NL80211_STA_INFO_TX_BITRATE.
   * @NL80211_STA_INFO_BSS_PARAM: current station's view of BSS, nested attribute
   *     containing info as possible, see &enum nl80211_sta_bss_param
   * @NL80211_STA_INFO_CONNECTED_TIME: time since the station is last connected
   * @NL80211_STA_INFO_STA_FLAGS: Contains a struct nl80211_sta_flag_update.
   * @NL80211_STA_INFO_BEACON_LOSS: count of times beacon loss was detected (u32)
   * @NL80211_STA_INFO_T_OFFSET: timing offset with respect to this STA (s64)
   * @NL80211_STA_INFO_LOCAL_PM: local mesh STA link-specific power mode
   * @NL80211_STA_INFO_PEER_PM: peer mesh STA link-specific power mode
   * @NL80211_STA_INFO_NONPEER_PM: neighbor mesh STA power save mode towards
   *	non-peer STA
   * @NL80211_STA_INFO_CHAIN_SIGNAL: per-chain signal strength of last PPDU
   *	Contains a nested array of signal strength attributes (u8, dBm)
   * @NL80211_STA_INFO_CHAIN_SIGNAL_AVG: per-chain signal strength average
   *	Same format as NL80211_STA_INFO_CHAIN_SIGNAL.
   * @NL80211_STA_EXPECTED_THROUGHPUT: expected throughput considering also the
   *	802.11 header (u32, kbps)
   * @__NL80211_STA_INFO_AFTER_LAST: internal
   * @NL80211_STA_INFO_MAX: highest possible station info attribute
   */
  ```

### Q：

1. 签到签退
   1. 发送请求？
   2. 轮询？
   3. 长连接，探测？
2. 大型组网方式下的改进，ac+ap/mesh？

