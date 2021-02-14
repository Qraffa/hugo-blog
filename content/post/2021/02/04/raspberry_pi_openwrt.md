---
title: "树莓派3B安装openwrt，安装usb无线网卡"
date: 2021-02-04T02:27:49+08:00
tags:
- 树莓派
- openwrt
categories: 
- 树莓派
- openwrt
---

## 树莓派3B安装openwrt，安装usb无线网卡

本文记录树莓派安装openwrt，安装usb无线网卡的操作过程

树莓派3B是捡垃圾捡来的，结果发现内置的无线网卡坏了，只好想办法买个usb无线网卡，给他装上驱动。

这里买的usb无线网卡是EDUP EP-N8508GS

<!--more-->

### 下载镜像

官网下载

对于树莓派3B使用的是bcm2710

https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi

http://downloads.openwrt.org/releases/19.07.6/targets/brcm2708/bcm2710/openwrt-19.07.6-brcm2708-bcm2710-rpi-3-ext4-factory.img.gz

### 刷入SD卡

SD卡格式化工具

[SD Card Formatter](https://www.sdcard.org/downloads/formatter/sd-memory-card-formatter-for-windows-download/)

系统刷入SD的工具

[balenaEtcher](https://github.com/balena-io/etcher)

### 启动树莓派

插入SD卡，插入电源，等待加载

### 连接树莓派

- 如果树莓派有显示器连接，那直接连上操作即可
- 如果是通过一条网线，将电脑与树莓派连在一起
- 如果树莓派是与路由器连接，电脑也与路由器连接
  - 树莓派默认设置网卡为静态地址192.168.1.1
  - 这是将本机电脑ip改成静态的192.168.1.2![image-20210204013855886](http://qiniu.qraffa.cn/image-20210204013855886.png)
  - 这时本机电脑与树莓派就在同一个网段了，可以互相ping通，因此可以ssh过去或者在浏览器中进入192.168.1.1

### 树莓派上网

启动dhcp获取ip

```bash
udhcpc -i br-lan
```

修改dns配置，添加dns解析服务器地址

```bash
vim /etc/resolve.conf

# 添加
nameserver 8.8.8.8
nameserver 8.8.4.4
```

PS：这个配置每次重启都会被重置，需要设置开机自启

#### 修改network配置

```bash
vim /etc/config/network
```

将之前的lan配置修改为

```bash
config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'dhcp'
```

以后每次启动都不用再手动配置就可连接上去了。

### opkg安装软件包

- 确定树莓派型号

  Raspberry Pi 3 Model B Rev 1.2

- 确定CPU架构型号

  ```bash
  root@OpenWrt:~# opkg print-architecture
  arch all 1
  arch noarch 1
  arch aarch64_cortex-a53 10
  ```

  树莓派3B CPU为ARM Cortex-A53

- 确保树莓派可以连上外网，比如

  ```bash
  ping baidu.com
  ```

  是有结果响应的

- 更新opkg软件包的软件源改成国内的

  - 进入luci web

  - 进入software配置![image-20210204015649911](http://qiniu.qraffa.cn/image-20210204015649911.png)

  - 修改软件源

    ![image-20210204015758255](http://qiniu.qraffa.cn/image-20210204015758255.png)

  ==！！！这里的软件源需要适配自己的openwrt版本和树莓派架构！！！==

  如果盲目直接复制可能出现无法安装等问题

  ```bash
  src/gz openwrt_core http://mirrors.ustc.edu.cn/openwrt/releases/19.07.6/targets/brcm2708/bcm2710/packages
  src/gz openwrt_base http://mirrors.ustc.edu.cn/openwrt/releases/19.07.6/packages/aarch64_cortex-a53/base
  src/gz openwrt_luci http://mirrors.ustc.edu.cn/openwrt/releases/19.07.6/packages/aarch64_cortex-a53/luci
  src/gz openwrt_packages http://mirrors.ustc.edu.cn/openwrt/releases/19.07.6/packages/aarch64_cortex-a53/packages
  src/gz openwrt_routing http://mirrors.ustc.edu.cn/openwrt/releases/19.07.6/packages/aarch64_cortex-a53/routing src/gz openwrt_telephony http://mirrors.ustc.edu.cn/openwrt/releases/19.07.6/packages/aarch64_cortex-a53/telephony
  ```

  - 设置好软件源配置后，更新列表，等待更新完成即可

    ![image-20210204020001463](http://qiniu.qraffa.cn/image-20210204020001463.png)

- 至此就可以安装需要的软件包了

### 安装usb wifi驱动

购买的usb wifi型号

EDUP EP-N8508GS

通过lsusb，可以看到是`RTL8188CUS`的

```bash
root@OpenWrt:/# lsusb
...
Bus 001 Device 004: ID 0bda:8176 Realtek Semiconductor Corp. RTL8188CUS 802.11n WLAN Adapter
...
```

- 安装rtl-8192cu驱动

  ![image-20210204020324829](http://qiniu.qraffa.cn/image-20210204020324829.png)

  安装时rtl-8192cu会自动安装其相关依赖

- 安装其他wifi工具

  ```bash
  iwinfo
  wireless-tools
  ```

- 安装好驱动之后，重启树莓派

- 重启树莓派后，可能在ifconfig中还是无法查看到，使用`iwinfo`查看是否有信息输出

  ```bash
  iwinfo wlan0 info
  ```

  或者使用`iwlist`让无线网卡扫描附近wifi，这样该设备就会启动起来，在ifconfig中就可以查看到了

  ```bash
  iwlist wlan0 scan
  ```

### 问题

#### 1. opkg 安装包提示未找到

如果提示安装包未找到，则表示

解决：更新opkg软件列表，最好更换国内镜像源

```bash
opkg update
```

#### 2. opkg安装包失败

在opkg安装软件包时，如果报如下的错误，则表示opkg软件源和当前的系统，cpu架构不匹配

```bash
Packages for xxx found, but incompatible with the architectures configured
```

解决：检查软件源是否是对应的版本

#### 3. 树莓派无法上网的问题

启动dhcp获取ip

```bash
udhcpc -i br-lan
```

修改dns配置，添加dns解析服务器地址

```bash
vim /etc/resolve.conf

# 添加
nameserver 8.8.8.8
nameserver 8.8.4.4
```

