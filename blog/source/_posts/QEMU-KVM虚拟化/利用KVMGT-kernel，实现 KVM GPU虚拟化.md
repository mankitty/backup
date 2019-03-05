# 利用KVMGT-kernel，实现 KVM GPU虚拟化
---

## 编译qemu-kvmgt
### 问题（1）
问题描述：ERROR: User requested feature sdl configure was not able to find it
解决方法：
yum list *SDL*
yum install SDL-devel.x86_64
### 问题（2）
问题描述：Preferred: Install the pixman devel package (any recent distro should have packages as Xorg needs pixman too)
解决方法：
yum install pixman-devel
### configure
./configure --prefix=/usr --enable-kvm --disable-xen --enable-debug-info --enable-debug --enable-sdl --enable-vhost-net --disable-debug-tcg --disable-werror --target-list=x86_64-softmmu
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