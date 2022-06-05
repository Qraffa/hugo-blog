---
title: "Iwinfo Cross Compile"
date: 2021-02-14T19:17:53+08:00
tags:
- openwrt
categories: 
- openwrt
---

## iwinfo交叉编译

由于做项目，需要使用到一些iwinfo中的方法函数，然后在openwrt上运行，因此尝试做个交叉编译

前提条件：编译了完整的openwrt，并且选了iwinfo软件包，在`make menuconfig`中选择了交叉编译的工具链。

<!--more-->

![](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/image-20210214193138471.png)

### 定位iwinfo包

一般来说，iwinfo软件包都在该目录下`openwrt-master/package/network/utils/iwinfo`，openwrt-master就是你下载到的openwrt源码存放的位置

在该目录下一般只有一个Makefile文件。

### 单独编译软件包

回到openwrt的顶级目录，这里就是回到`openwrt-master`目录下

运行命令，编译iwinfo

```bash
make package/network/utils/iwinfo/compile V=1
```

得到编译过程

```bash
 make[1] package/network/utils/iwinfo/compile
 make[2] -C package/libs/toolchain compile
 make[2] -C package/libs/libjson-c compile
 make[2] -C package/utils/lua compile
 make[2] -C package/libs/libubox compile
 make[2] -C package/system/ubus compile
 make[2] -C package/system/uci compile
 make[2] -C package/libs/libnl-tiny compile
 make[2] -C package/network/utils/iwinfo compile
```

### 查看iwinfo的详细编译过程

需要手动编译查看详细的编译过程

```bash
make -C package/network/utils/iwinfo compile
```

执行以上会提示`/rules.mk: 没有那个文件或目录`，这是需要制定以下TOPDIR

TOPDIR修改成你自己的openwrt目录

```bash
make -C package/network/utils/iwinfo TOPDIR=/home/qraffa/wrt/openwrt-master compile
```

接下来你将看到一大堆编译信息输出，这些信息就是交叉编译iwinfo时的详细过程，去掉一些不重要的信息，我们这里主要是要关注编译的参数

```bash
CFLAGS="-Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE  -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -I/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/usr/include -I/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/include/fortify -I/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/include " CXXFLAGS="-Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE  -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -I/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/usr/include -I/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/include/fortify -I/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/include " LDFLAGS="-L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/lib -znow -zrelro " make -j1 -C /home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5/. AR="aarch64-openwrt-linux-musl-gcc-ar" AS="aarch64-openwrt-linux-musl-gcc -c -Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE" LD=aarch64-openwrt-linux-musl-ld NM="aarch64-openwrt-linux-musl-gcc-nm" CC="aarch64-openwrt-linux-musl-gcc" GCC="aarch64-openwrt-linux-musl-gcc" CXX="aarch64-openwrt-linux-musl-g++" RANLIB="aarch64-openwrt-linux-musl-gcc-ranlib" STRIP=aarch64-openwrt-linux-musl-strip OBJCOPY=aarch64-openwrt-linux-musl-objcopy OBJDUMP=aarch64-openwrt-linux-musl-objdump SIZE=aarch64-openwrt-linux-musl-size CROSS="aarch64-openwrt-linux-musl-" ARCH="aarch64" FPIC="-fpic" CFLAGS="-Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE" LDFLAGS="-L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/lib -znow -zrelro" BACKENDS="   nl80211" ;

rm -f *.o libiwinfo.so iwinfo.so iwinfo

aarch64-openwrt-linux-musl-gcc -Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211 -fpic -c -o iwinfo_utils.o iwinfo_utils.c

aarch64-openwrt-linux-musl-gcc -Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211 -fpic -c -o iwinfo_wext.o iwinfo_wext.c

aarch64-openwrt-linux-musl-gcc -Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211 -fpic -c -o iwinfo_wext_scan.o iwinfo_wext_scan.c

aarch64-openwrt-linux-musl-gcc -Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211 -fpic -c -o iwinfo_lib.o iwinfo_lib.c

aarch64-openwrt-linux-musl-gcc -Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211 -fpic -c -o iwinfo_nl80211.o iwinfo_nl80211.c

aarch64-openwrt-linux-musl-gcc -Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211 -fpic -c -o iwinfo_lua.o iwinfo_lua.c

aarch64-openwrt-linux-musl-gcc -Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211 -fpic -c -o iwinfo_cli.o iwinfo_cli.c

aarch64-openwrt-linux-musl-gcc -luci -lubox -lubus -L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/lib -znow -zrelro -shared -lnl-tiny -o libiwinfo.so iwinfo_utils.o iwinfo_wext.o iwinfo_wext_scan.o iwinfo_lib.o iwinfo_nl80211.o

aarch64-openwrt-linux-musl-gcc -luci -lubox -lubus -L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/lib -znow -zrelro -shared -L. -liwinfo -llua -o iwinfo.so iwinfo_lua.o

aarch64-openwrt-linux-musl-gcc -luci -lubox -lubus -L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/lib -znow -zrelro -L. -liwinfo -lnl-tiny -o iwinfo iwinfo_cli.o
```

`aarch64-openwrt-linux-musl-gcc`这东西是用来做交叉编译的编译器。等会我们做交叉编译也同样需要用到该工具。



### 找到编译出来的iwinfo位置

`/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5`

一般都位于openwrt目录下的`build_dir`目录，`target-xxx`这个和你编译openwrt选择的目标平台有关，`libiwinfo-xxx`这里的xxx和你的iwinfo源码git版本有关。

### 自定义软件

这里做一个输出bssid和name的简单demo

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

### 定位交叉工具链

`aarch64-openwrt-linux-musl-gcc`交叉编译工具，直接运行是会提示commad not found

在编译完整个openwrt系统后，在target目录下可以找到编译工具链的包，该目录也就是你拿编译出来的镜像文件的位置。

在该目录下`/home/qraffa/wrt/openwrt-master/bin/targets/bcm27xx/bcm2710`

会存在一个包`openwrt-toolchain-bcm27xx-bcm2710_gcc-8.4.0_musl.Linux-x86_64.tar.bz2`，这里的名称也是和你选择的系统架构有关。

将该包解压到某个目录下。

解压后，在对应目录的bin目录下找到`aarch64-openwrt-linux-gcc`

### 交叉编译

1. 将自定义软件源码复制到iwinfo编译出来的位置，比如我这里是`a.c`

2. 编译第一步

   ```bash
   /home/qraffa/iw/toolchain/toolchain/bin/aarch64-openwrt-linux-musl-gcc -Os -pipe -mcpu=cortex-a53 -fno-caller-saves -fno-plt -fhonour-copts -Wno-error=unused-but-set-variable -Wno-error=unused-result -fmacro-prefix-map=/home/qraffa/wrt/openwrt-master/build_dir/target-aarch64_cortex-a53_musl/libiwinfo-2020-06-03-2faa20e5=libiwinfo-2020-06-03-2faa20e5 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include/libnl-tiny -I/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/include -D_GNU_SOURCE -std=gnu99 -fstrict-aliasing -Iinclude -DUSE_NL80211 -fpic -c -o a.o a.c
   ```

   这里就是copy的`iwinfo_utils.c`的编译过程，需要改成你自己对应的。

   然后还需要把`/home/qraffa/iw/toolchain/toolchain/bin/aarch64-openwrt-linux-musl-gcc`改成你的交叉编译链的位置。

3. 编译第二步

   ```bash
   /home/qraffa/iw/toolchain/toolchain/bin/aarch64-openwrt-linux-musl-gcc -luci -lubox -lubus -L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/target-aarch64_cortex-a53_musl/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/usr/lib -L/home/qraffa/wrt/openwrt-master/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/lib -znow -zrelro -L. -liwinfo -lnl-tiny -o a a.o
   ```

   这里就是copy的`iwinfo`的编译过程，也就是上面详细编译过程的最后一步，需要改成你自己对应的。

   然后还需要把`/home/qraffa/iw/toolchain/toolchain/bin/aarch64-openwrt-linux-musl-gcc`改成你的交叉编译链的位置。

4. 编译完成

### 测试运行

将上面编译出来的可执行文件`a`，scp到openwrt上测试

wlan0改成你自己的无线网卡的名称

````bash
root@OpenWrt:~# ./a wlan0
nl80211
E8:4E:06:7C:4E:0C
````

