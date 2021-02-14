---
title: "Iwinfo Compile"
date: 2021-02-15T01:07:06+08:00
tags:
- openwrt
categories: 
- openwrt
---

## iwinfo编译

尝试在openwrt上，自己编译iwinfo，并引用iwinfo的库编译自己的软件

<!--more-->

### 获取源码

[git://git.openwrt.org/project/iwinfo.git](git://git.openwrt.org/project/iwinfo.git)

切换到2020-06-03的版本

```bash
git checkout 2faa20e5e9d107b97e393a4eb458370e80b4d720
```

如果这里不切换版本，那么后面编译出来的.so文件可能会报错

`iwinfo_backend: symbol not found`

### 添加依赖库

#### include

没添加include的相关头文件，会报错`fatal error: xxx.h: No such file or directory`

该include一般在`xxx/staging_dir/target-aarch64_cortex-a53_musl/usr/include`

xxx就是你拉的openwrt源码所在的位置

target-aarch64_cortex-a53_musl对应于你所编译选择的架构

需要添加的文件：

- libnl-tiny
- libubox
- uci.h
- uci_config.h
- libubus.h
- ubus_common.h
- ubusmsg.h
- lua.h
- luaconf.h
- lnum_config.h
- lualib.h
- lauxlib.h

可以将以上文件添加到openwrt的某个目录下，比如我这里添加到的是`/root/wrt/include`

#### lib

没添加相关动态库，会报错`cannot find -lxxx`

该lib库一般在`xxx/staging_dir/target-aarch64_cortex-a53_musl/usr/lib`

与include类似

需要添加的文件：

- liblua.so.5.1.5
  - 该lua添加后，还需要在添加的目录下做个软连接` ln -s liblua.so.5.1.5 liblua.so`
- libuci.so
- libubox.so
- libubus.so

### 编译iwinfo

添加好以上依赖，将iwinfo源码scp到openwrt上，就可以开始编译了

编译命令

```bash
make CFLAGS="-Iinclude -I/root/wrt/include -I/root/wrt/include/libnl-tiny -fpic -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing" LDFLAGS="-L/root/wrt/lib -lnl-tiny" BACKENDS=nl80211
```

- `-I/root/wrt/include`：指向你添加的include的目录
- `-L/root/wrt/lib`：指向你添加的lib的目录

```bash
root@OpenWrt:~/wrt/iwinfo# make CFLAGS="-Iinclude -I/root/wrt/include -I/root/wrt/include/libnl-tiny -fpic -D_GNU_SOURCE -std=gnu99 -fs
trict-aliasing" LDFLAGS="-L/root/wrt/lib -lnl-tiny" BACKENDS=nl80211
rm -f *.o libiwinfo.so iwinfo.so iwinfo
cc -Iinclude -I/root/wrt/include -I/root/wrt/include/libnl-tiny -fpic -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211  -c -o iwinfo_utils.o iwinfo_utils.c
cc -Iinclude -I/root/wrt/include -I/root/wrt/include/libnl-tiny -fpic -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211  -c -o iwinfo_wext.o iwinfo_wext.c
cc -Iinclude -I/root/wrt/include -I/root/wrt/include/libnl-tiny -fpic -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211  -c -o iwinfo_wext_scan.o iwinfo_wext_scan.c
cc -Iinclude -I/root/wrt/include -I/root/wrt/include/libnl-tiny -fpic -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211  -c -o iwinfo_lib.o iwinfo_lib.c
cc -Iinclude -I/root/wrt/include -I/root/wrt/include/libnl-tiny -fpic -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211  -c -o iwinfo_nl80211.o iwinfo_nl80211.c
cc -Iinclude -I/root/wrt/include -I/root/wrt/include/libnl-tiny -fpic -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211  -c -o iwinfo_lua.o iwinfo_lua.c
cc -Iinclude -I/root/wrt/include -I/root/wrt/include/libnl-tiny -fpic -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211  -c -o iwinfo_cli.o iwinfo_cli.c
cc -luci -lubox -lubus -L/root/wrt/lib -lnl-tiny -shared -lnl-tiny -o libiwinfo.so iwinfo_utils.o iwinfo_wext.o iwinfo_wext_scan.o iwinfo_lib.o iwinfo_nl80211.o
cc -luci -lubox -lubus -L/root/wrt/lib -lnl-tiny -shared -L. -liwinfo -llua -o iwinfo.so iwinfo_lua.o
cc -luci -lubox -lubus -L/root/wrt/lib -lnl-tiny -L. -liwinfo -lnl-tiny -o iwinfo iwinfo_cli.o
```

编译完成。测试iwinfo是否正常使用

```bash
root@OpenWrt:~/wrt/iwinfo# ./iwinfo
wlan0     ESSID: "OpenWrt"
          Access Point: E8:4E:06:7C:4E:0C
          Mode: Master  Channel: 11 (2.462 GHz)
          Tx-Power: 20 dBm  Link Quality: 39/70
          Signal: -71 dBm  Noise: unknown
          Bit Rate: 72.2 MBit/s
          Encryption: none
          Type: nl80211  HW Mode(s): 802.11bgn
          Hardware: unknown [Generic MAC80211]
          TX power offset: unknown
          Frequency offset: unknown
          Supports VAPs: no  PHY name: phy0
```

### 编译自定义软件

需要编译的源码

```c
#include <stdio.h>
#include "iwinfo.h"

static char * print_bssid(const struct iwinfo_ops *iw, const char *ifname)
{
	static char buf[18] = { 0 };

	if (iw->bssid(ifname, buf))
		snprintf(buf, sizeof(buf), "00:00:00:00:00:00");

	return buf;
}

int main(int argc, char **argv)
{
  char* ifname = argv[1];
  struct iwinfo_ops* iw;
  iw = iwinfo_backend(argv[1]);
  if (!iw)
  {
    fprintf(stderr, "No such wireless device: %s\n", argv[1]);
  }
  printf("%s\n",iw->name);
  // iw->ssid(ifname, buf);
  printf("%s\n", print_bssid(iw,argv[1]));
  return 0;
}
```

编译命令：

```bash
gcc -I/root/wrt/include/libnl-tiny -I/root/wrt/include -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211 -fpic -c -o a.o a.c
gcc -luci -lubox -lubus -L/root/wrt/lib -L. -liwinfo -lnl-tiny -o a a.o
```

编译完成。测试能否正常使用

```bash
root@OpenWrt:~/wrt/iwinfo# ./a wlan0
nl80211
E8:4E:06:7C:4E:0C
```

