---
title: OpenWRT中添加驱动模块
date: 2018-07-04 09:12:08
tags: [OpenWRT,驱动]
description: 介绍如何在OpenWRT中添加驱动模块，学习《LINUX设备驱动程序-第三版》
---
### 0. 准备工作
硬件：主芯片 MT7628NN，Flash 16M，内存 64M  
软件：基于 MTK 原厂 SDK，OpenWrt v3.4，mtk-openwrt-sdk-20160324-8f8e4f1e.tar.bz2  
文档：《Linux设备驱动程序 第三版》

### 1. 添加模块目录
在目录 `package` 中创建模块目录 `helloworld`，目录结构如下：
```
$ tree package/kernel/helloworld/
package/kernel/helloworld/
├── Makefile
└── src
    ├── helloworld.c
    └── Makefile

1 directory, 3 files
$
```
目录 `helloworld` 中的文件 `Makefile` 内容如下：
```
$ cat package/kernel/helloworld/Makefile
#
# Copyright (C) 2008-2012 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=helloworld
PKG_RELEASE:=1.0

include $(INCLUDE_DIR)/package.mk

define KernelPackage/helloworld
  SUBMENU:=Other modules
  TITLE:=helloworld for test
  FILES:=$(PKG_BUILD_DIR)/helloworld.ko
endef

define Build/Prepare
        mkdir -p $(PKG_BUILD_DIR)
        $(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Build/Compile
        $(MAKE) -C "$(LINUX_DIR)" V=1 \
        CROSS_COMPILE="$(TARGET_CROSS)" \
        ARCH="$(LINUX_KARCH)" \
        SUBDIRS="$(PKG_BUILD_DIR)" \
        modules
endef

$(eval $(call KernelPackage,helloworld))
$
```
目录 `helloworld/src` 中的文件 `helloworld.c` 内容如下：
```
$ cat package/kernel/helloworld/src/helloworld.c 
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");

static int hello_init(void)
{
    printk(KERN_ALERT "Hello world\n");
    return 0;
}

static void hello_exit(void)
{
    printk(KERN_ALERT "Goodbye, cruel world\n");
}

module_init(hello_init);
module_exit(hello_exit);
$ 
```
目录 `helloworld/src` 中的文件 `Makefile` 内容如下：
```
$ cat package/kernel/helloworld/src/Makefile 
obj-m += helloworld.o
$ 
```
完成上诉目录结构建立，接下来进行编译。
### 2. 编译模块
输入命令 `make menuconfig`，打开编译选项 *kmod-helloworld*
```
$ make menuconfig
Kernel modules  --->
    Other modules  --->
        < > kmod-helloworld...................................... helloworld for test
```
输入一下回车，改为 M
```
        <M> kmod-helloworld...................................... helloworld for test
```
输入命令 `make package/kernel/helloworld/compile V=s`，编译 *helloworld* 模块，等待编译成功
```
$ make package/kernel/helloworld/compile V=s
```
成功后可以在目录 `build_dir` 下找到 *helloworld.ko* 文件，可以在目录 `bin` 下找到 *ipk* 文件
```
$ ls build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7628/helloworld/helloworld.ko 
build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7628/helloworld/helloworld.ko
$
$ ls bin/ramips/packages/base/kmod-helloworld_3.10.14-1.0_ramips_24kec.ipk 
bin/ramips/packages/base/kmod-helloworld_3.10.14-1.0_ramips_24kec.ipk
```
ko 文件是 Linux 原生的驱动模块文件，可以使用 Linux 原生命令进行安装  
ipk 文件是 OpenWRT 特有的软件包，需要配合 OpenWRT 中的命令进行安装

### 3. 文件传入子板
接下来需要将 ko 文件和 ipk 文件传入子板  
配置 PC 和子板的 IP

|     | PC         | 子板       |
| --- | ---------- | ---------- |
| IP  | 10.7.10.10 | 10.7.10.15 |

PC 的 IP 配置方式省略，重点介绍如何在 OpenWRT 中配置子板 IP  
输入命令 `uci show network`，查看子板网络配置
```
root@OpenWrt:/# uci show network
network.loopback=interface
network.loopback.ifname=lo
network.loopback.proto=static
network.loopback.ipaddr=127.0.0.1
network.loopback.netmask=255.0.0.0
network.globals=globals
network.globals.ula_prefix=fd74:5f6a:5a8d::/48
network.lan=interface
network.lan.ifname=eth0.1
network.lan.force_link=1
network.lan.type=bridge
network.lan.proto=static
network.lan.netmask=255.255.255.0
network.lan.ip6assign=60
network.lan.ipaddr=192.168.1.1
network.wan=interface
network.wan.ifname=eth0.2
network.wan.proto=dhcp
network.wan.enabled=1
network.wl_wan=interface
network.wl_wan.ifname=apcli0
network.wl_wan.proto=dhcp
network.wl_wan.enabled=0
network.@switch[0]=switch
network.@switch[0].name=switch0
network.@switch[0].reset=1
network.@switch[0].enable_vlan=1
network.@switch_vlan[0]=switch_vlan
network.@switch_vlan[0].device=switch0
network.@switch_vlan[0].vlan=1
network.@switch_vlan[0].ports=0 1 2 3 5 6t
network.@switch_vlan[1]=switch_vlan
network.@switch_vlan[1].device=switch0
network.@switch_vlan[1].vlan=2
network.@switch_vlan[1].ports=4 6t
root@OpenWrt:/# 
```
可以看到 `network.lan.ipaddr=192.168.1.1`，当前子板 IP 为 192.168.1.1  
输入命令 `uci set network.lan.ipaddr=10.7.10.15`，修改子板 IP 为 10.7.10.15（尚未生效）  
输入命令 `uci commit network`  保存当前配置，重启有效
输入命令 `/etc/init.d/network restart` 重启网络脚本，使配置生效
```
root@OpenWrt:/# uci set network.lan.ipaddr=10.7.10.15
root@OpenWrt:/# uci commit network
root@OpenWrt:/# /etc/init.d/network restart
```

待脚本执行结束，输入命令 `ifconfig`，看到 *br-lan* 的 IP 已经修改成功
```
root@OpenWrt:/# ifconfig br-lan
br-lan    Link encap:Ethernet  HWaddr 00:0C:43:E1:76:29  
          inet addr:10.7.10.15  Bcast:10.7.10.255  Mask:255.255.255.0
          inet6 addr: fd74:5f6a:5a8d::1/60 Scope:Global
          inet6 addr: fe80::20c:43ff:fee1:7629/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:122 errors:0 dropped:0 overruns:0 frame:0
          TX packets:28 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:7764 (7.5 KiB)  TX bytes:3341 (3.2 KiB)

root@OpenWrt:/# 
```
输入命令 `ping 10.7.10.10`，ping PC，看到可以 ping 通，说明网络已经通畅
```
root@OpenWrt:/# 
确定可以 ping 通 10.7.10.10
root@OpenWrt:/# ping 10.7.10.10
PING 10.7.10.10 (10.7.10.10): 56 data bytes
64 bytes from 10.7.10.10: seq=0 ttl=64 time=1.160 ms
64 bytes from 10.7.10.10: seq=1 ttl=64 time=0.656 ms
64 bytes from 10.7.10.10: seq=2 ttl=64 time=0.720 ms
^C
--- 10.7.10.10 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.656/0.845/1.160 ms
root@OpenWrt:/# 
```

通过 tftp 获取 PC 上的 ko 文件和 ipk 文件，PC 端 tftp server 通过 tftpd32.exe 搭建
输入如下命令，将 *ko* 文件和 *ipk* 文件传入子板
```
root@OpenWrt:/# mkdir tmp/ko
root@OpenWrt:/# cd tmp/ko/
root@OpenWrt:/tmp/ko# tftp -gr helloworld.ko  10.7.10.10 
root@OpenWrt:/tmp/ko# ls
helloworld.ko
root@OpenWrt:/tmp/ko# tftp -gr kmod-helloworld_3.10.14-1.0_ramips_24kec.ipk  10.7.10.10
root@OpenWrt:/tmp/ko# ls
helloworld.ko
kmod-helloworld_3.10.14-1.0_ramips_24kec.ipk
root@OpenWrt:/tmp/ko#
```
### 4. 安装驱动
#### 4.1 安装ko文件
ko 文件通过 Linux 原生命令 insmod、rmmod 和 lsmod 分别进行安装、卸载和查看  
输入命令 `insmod helloworld.ko`，安装驱动模块，可以看到模块初始化的打印
```
root@OpenWrt:/tmp/ko# insmod helloworld.ko 
[ 3815.112000] Hello world
root@OpenWrt:/tmp/ko# 
```
输入命令 `lsmod`，查看已经安装的驱动模块，可以看到模块 *helloworld*
```
root@OpenWrt:/tmp/ko# lsmod 
 ... ...
em_u32                   576  0 
fat                    48319  1 vfat
helloworld               626  0 
ifb                     3104  0 
ip6_tables              9649  3 ip6table_raw
 ... ...
root@OpenWrt:/tmp/ko# 
```
输入命令 `rmmod helloworld`，卸载驱动模块，可以看到模块卸载的打印
```
root@OpenWrt:/tmp/ko# rmmod helloworld
[ 3822.944000] Goodbye, cruel world
root@OpenWrt:/tmp/ko# 
```
#### 4.2 安装ipk文件
ipk 文件通过 OpenWRT 命令 opkg 安装和卸载  
输入命令 `opkg install kmod-helloworld_3.10.14-1.0_ramips_24kec.ipk`，安装 ipk 文件
```
root@OpenWrt:/tmp/ko# opkg install kmod-helloworld_3.10.14-1.0_ramips_24kec.ipk 
Installing kmod-helloworld (3.10.14-1.0) to root...
Configuring kmod-helloworld.
root@OpenWrt:/tmp/ko# 
```
输入命令 `opkg list`，可以看到 *kmod-helloworld - 3.10.14-1.0* 已经安装
```
root@OpenWrt:/tmp/ko# opkg list
 ... ...
kmod-fs-vfat - 3.10.14-1
kmod-gre - 3.10.14-1
kmod-helloworld - 3.10.14-1.0
kmod-hw_wdg - 3.10.14-1
kmod-ifb - 3.10.14-1
 ... ...
root@OpenWrt:/tmp/ko#
```
查看目录 `/lib/modules/3.10.14/`，发现多了一个文件 *helloworld.ko*，ko 文件的安装方式上文已经介绍
```
root@OpenWrt:/tmp/ko# ls /lib/modules/3.10.14/
 ... ...
em_u32.ko
fat.ko
helloworld.ko
ifb.ko
ip6_tables.ko
 ... ...
root@OpenWrt:/tmp/ko# 
```
输入命令 `opkg remove kmod-helloworld - 3.10.14-1.0`，卸载 ipk 文件
```
root@OpenWrt:/tmp/ko# opkg remove kmod-helloworld - 3.10.14-1.0
Removing package kmod-helloworld from root...
root@OpenWrt:/tmp/ko# 
```

### 5、总结
至此，我们已经可以在 OpenWRT 中编译和安装驱动模块了，一个好的开始！






