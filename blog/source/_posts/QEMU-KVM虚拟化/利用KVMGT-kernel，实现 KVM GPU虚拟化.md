---
title: KVM GPU虚拟化
tags:
categories: 虚拟化
---

## 编译qemu-kvmgt

yum install gcc gcc-c++ libtool -y

yum install SDL2-devel pulseaudio-libs-devel libepoxy-devel

### 问题（1）

问题描述：ERROR: User requested feature sdl configure was not able to find it
解决方法：
yum list *SDL*
yum install SDL-devel.x86_64

### 问题（2）
问题描述：Preferred: Install the pixman devel package (any recent distro should have packages as Xorg needs pixman too)
解决方法：
yum install pixman-devel

### 问题（3）
问题描述：ERROR: User requested feature gtk
       configure was not able to find it.
       Install gtk3-devel
解决方法：
yum install gtk3-devel gtk2-devel

### 问题（4）
问题描述：
ERROR: User requested feature opengl
       configure was not able to find it.
       Please install opengl (mesa) devel pkgs: epoxy libdrm gbm
解决方法：
yum install mesa-libGL

### 问题（5）
问题描述：ERROR: zlib check failed
       Make sure to have the zlib libs and headers installed
解决方法：
yum install -y glib*
yum install zlib*

ERROR: User requested feature spice
       configure was not able to find it.
       Install spice-server(>=0.12.0) and spice-protocol(>=0.12.3) devel
yum install spice-server spice-server-devel
yum install spice-protocol

ERROR: User requested feature libusb
       configure was not able to find it.
       Install libusb devel >= 1.0.13

ERROR: User requested feature libusb
       configure was not able to find it.
       Install libusb devel >= 1.0.13
yum install libusb



yum install mesa-libgbm mesa-libgbm-devel

yum install libdrm libdrm-devel

yum install gtkglext-devel gtkglext-libs libreoffice-ogltrans

### configure
./configure --prefix=/usr --enable-kvm --disable-xen --enable-libusb  --enable-debug-info --enable-debug --enable-sdl --enable-vhost-net --enable-spice --disable-debug-tcg  --enable-opengl --enable-gtk --target-list=x86_64-softmmu

yum install mesa-libgbm mesa-libgbm-devel -y
yum install libdrm libdrm-devel -y



yum install mesa-libEGL-devel

### 问题（5）
问题描述：
Could not access KVM kernel module: No such file or directory
解决方法：
modprobe kvm-intel
### 升级内核
grub2-mkconfig -o /boot/grub2/grub.cfg

## 编译内核（CENTOS 6.5）
### 基本软件
yum install git ncurses-devel bison flex elfutils-libelf-devel
### 安装编译安装需要的包组
yum groupinstall "development tools"
### 问题（1）
 HOSTLD  scripts/kconfig/mconf
/usr/bin/ld: scripts/kconfig/lxdialog/checklist.o: undefined reference to symbol 'acs_map'
/usr/bin/ld: note: 'acs_map' is defined in DSO /lib64/libtinfo.so.5 so try adding it to the linker command line
/lib64/libtinfo.so.5: could not read symbols: Invalid operation
collect2: error: ld returned 1 exit status
make[1]: *** [scripts/kconfig/mconf] Error 1
make: *** [menuconfig] Error 2

解决方法：
编辑 linux-3.17.4/scripts/kconfig/Makefile，加入：
HOSTLOADLIBES_mconf   = $(shell $(CONFIG_SHELL) $(check-lxdialog) -ldflags $(HOSTCC)) -ltinfo
### 问题（2）
scripts/sign-file.c:25:30: fatal error: openssl/opensslv.h: No such file or directory
#include <openssl/opensslv.h>
解决方法：
yum install openssl-devel